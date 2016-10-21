Django has its own implementation for **reset/forgot password** for its admin site. We are going to use that piece of code as reference to implement similar feature for a non admin-site authentication page. Although there are tons of good packages which will allow user to use their password reseting system. But if the system isn't too complex and doesn't need such authentication plugins, then reusing the django's very own implementation can be a good option.

Class based view is going to be used instead of method based view(for no particular reason, so using either of them is alright.). And please read the comments of the example codes for better understanding of implementation.


This implementation is going to divided into two parts. First part is sending an email with reset url, and the Second part is clicking the reset url attached in email and entering new password for reset completion.

Before starting anything, lets look at the django's reset/forgot password's implementation in *django/contrib/auth/forms.py* (<a href='https://github.com/django/django/blob/89559bcfb096ccc625e0e9ab41e2136fcb32a514/django/contrib/auth/forms.py'>source</a>) and *django/contrib/auth/views.py* (<a href='https://github.com/django/django/blob/89559bcfb096ccc625e0e9ab41e2136fcb32a514/django/contrib/auth/views.py'>source</a>).

##Implemetation of sending an email for forgot password with reset url

First need to configure smtp/email configuration so the system can send email. Gmail's SMTP service is going to be used here.

    EMAIL_USE_TLS = True
    DEFAULT_FROM_EMAIL = 'test@gmail.com'
    SERVER_EMAIL = 'test@gmail.com'
    EMAIL_HOST = 'smtp.gmail.com'
    EMAIL_PORT = 587
    EMAIL_HOST_USER = 'test@gmail.com'
    EMAIL_HOST_PASSWORD = 'test123##'
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

Now we are going to make a <b>reset password form</b> where we are going to add an text field which will take either username or email address associated with the corresponding user.

    from django import forms

    class PasswordResetRequestForm(forms.Form):
        email_or_username = forms.CharField(label=("Email Or Username"), max_length=254)


We are going to make a <b>view</b> which will check the input email/username and send an email to user's email address(implementation reference: <a href='https://github.com/django/django/blob/89559bcfb096ccc625e0e9ab41e2136fcb32a514/django/contrib/auth/forms.py#L235'>source</a>).


        from django.contrib.auth.tokens import default_token_generator
        from django.utils.encoding import force_bytes
        from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
        from django.template import loader
        from django.core.validators import validate_email
        from django.core.exceptions import ValidationError
        from django.core.mail import send_mail
        from settings import DEFAULT_FROM_EMAIL
        from django.views.generic import *
        from utils.forms.reset_password_form import PasswordResetRequestForm
        from django.contrib import messages
        from django.contrib.auth.models import User
        from django.db.models.query_utils import Q

        class ResetPasswordRequestView(FormView):
            template_name = "account/test_template.html"    #code for template is given below the view's code
            success_url = '/account/login'
            form_class = PasswordResetRequestForm

            @staticmethod
            def validate_email_address(email):
            '''
            This method here validates the if the input is an email address or not. Its return type is boolean, True if the input is a email address or False if its not.
            '''
                try:
                    validate_email(email)
                    return True
                except ValidationError:
                    return False

            def post(self, request, *args, **kwargs):
            '''
            A normal post request which takes input from field "email_or_username" (in ResetPasswordRequestForm). 
            '''
                form = self.form_class(request.POST)
                if form.is_valid():
                    data= form.cleaned_data["email_or_username"]
                if self.validate_email_address(data) is True:                 #uses the method written above
                    '''
                    If the input is an valid email address, then the following code will lookup for users associated with that email address. If found then an email will be sent to the address, else an error message will be printed on the screen.
                    '''
                    associated_users= User.objects.filter(Q(email=data)|Q(username=data))
                    if associated_users.exists():
                        for user in associated_users:
                                c = {
                                    'email': user.email,
                                    'domain': request.META['HTTP_HOST'],
                                    'site_name': 'your site',
                                    'uid': urlsafe_base64_encode(force_bytes(user.pk)),
                                    'user': user,
                                    'token': default_token_generator.make_token(user),
                                    'protocol': 'http',
                                    }
                                subject_template_name='registration/password_reset_subject.txt' 
                                # copied from django/contrib/admin/templates/registration/password_reset_subject.txt to templates directory
                                email_template_name='registration/password_reset_email.html'    
                                # copied from django/contrib/admin/templates/registration/password_reset_email.html to templates directory
                                subject = loader.render_to_string(subject_template_name, c)
                                # Email subject *must not* contain newlines
                                subject = ''.join(subject.splitlines())
                                email = loader.render_to_string(email_template_name, c)
                                send_mail(subject, email, DEFAULT_FROM_EMAIL , [user.email], fail_silently=False)
                        result = self.form_valid(form)
                        messages.success(request, 'An email has been sent to ' + data +". Please check its inbox to continue reseting password.")
                        return result
                    result = self.form_invalid(form)
                    messages.error(request, 'No user is associated with this email address')
                    return result
                else:
                    '''
                    If the input is an username, then the following code will lookup for users associated with that user. If found then an email will be sent to the user's address, else an error message will be printed on the screen.
                    '''
                    associated_users= User.objects.filter(username=data)
                    if associated_users.exists():
                        for user in associated_users:
                            c = {
                                'email': user.email,
                                'domain': 'example.com', #or your domain
                                'site_name': 'example',
                                'uid': urlsafe_base64_encode(force_bytes(user.pk)),
                                'user': user,
                                'token': default_token_generator.make_token(user),
                                'protocol': 'http',
                                }
                            subject_template_name='registration/password_reset_subject.txt'
                            email_template_name='registration/password_reset_email.html'
                            subject = loader.render_to_string(subject_template_name, c)
                            # Email subject *must not* contain newlines
                            subject = ''.join(subject.splitlines())
                            email = loader.render_to_string(email_template_name, c)
                            send_mail(subject, email, DEFAULT_FROM_EMAIL , [user.email], fail_silently=False)
                        result = self.form_valid(form)
                        messages.success(request, 'Email has been sent to ' + data +"'s email address. Please check its inbox to continue reseting password.")
                        return result
                    result = self.form_invalid(form)
                    messages.error(request, 'This username does not exist in the system.')
                    return result
                messages.error(request, 'Invalid Input')
                return self.form_invalid(form)

As you see above, the code is fairly simple(although it looks long). An encoded user id has been generated here using **urlsafe__base64___encode(force___bytes(user.pk))** and a token by using **default___token___generator.make___token(user)**. This user id is going to be used later to get the user, the token will be used for checking validity of the url for that user and both the token and the user id is going to be used as unique reference for reset password url. **c** is a dictionary which has user id, token and other related data etc. This dictionary is going to be blent with the template `registration/password_reset_email.html` and send to the user's email address.

For displaying messages(if you are using messages framework of django-1.7, details: <a href='https://docs.djangoproject.com/en/dev/ref/contrib/messages/#displaying-messages'>source</a>), add this piece of code in your template:


        {# test template #}
        <!-- code for displaying success or error message in template -->
        {% if messages %}
        <ul class="messages">
            {% for message in messages %}
            <li>{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
            {% endfor %}
            {% endif %}
        </ul>

        <!-- Form rendering code for template -->
        <form action="" method="post">

            {% csrf_token %}
            {{ form.as_p }}

            <input type="submit" value="Submit" />
        </form>

Two more things before wrapping up sending email part. One, making a <b>url</b> for using this view.


    urlpatterns = patterns('',
                           url(r'^admin/', include(admin.site.urls)),
                           # url(r'^account/reset_password_confirm/(?P<uidb64>[0-9A-Za-z]+)-(?P<token>.+)/$', PasswordResetConfirmView.as_view(),name='reset_password_confirm'), 
                           # PS: url above is going to used for next section of implementation.
                           url(r'^account/reset_password', ResetPasswordRequestView.as_view(), name="reset_password"),  
                           )


Two, editing the template of **registration/password_reset_email.html** or else you will get errors.


    {% load i18n %}{% autoescape off %}
    {% blocktrans %}You're receiving this email because you requested a password reset for your user account at {{ site_name }}.{% endblocktrans %}

    {% trans "Please go to the following page and choose a new password:" %}
        {% block reset_link %}
            {{ domain }}{% url 'reset_password_confirm' uidb64=uid token=token %} 
            <!--This is the only change from ` django/contrib/admin/templates/registration/password_reset_subject.html`. the url name is commented out in urls.py section. The view associated with the url is going to described later in this post. -->
        {% endblock %}
    {% trans "Your username, in case you've forgotten:" %} {{ user.get_username }}

    {% trans "Thanks for using our site!" %}

    {% blocktrans %}The {{ site_name }} team{% endblocktrans %}

    {% endautoescape %}

Now run the server and you will see forms like the screen shots below: (This screenshots look cool because django adminsite's js/css have been used here.)
(**Image sequence is according the implementation flow)

###Rendered template from PasswordResetRequestForm form

<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/reset-password.png">

###Rendered template from PasswordResetRequestForm form with error messages

<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/invalid-username.png">


###Rendered template of login form with sent email confirmation message

<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/reset-mail-sent.png">

###Sent Email look
<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/reset-email.png">


##Implemetation of clicking the reset url and entering new password for reset completation.

First, lets write a <b>form</b> which will have two fields new password and retype password field. 


    class SetPasswordForm(forms.Form):
        """
        A form that lets a user change set their password without entering the old
        password
        """
        error_messages = {
            'password_mismatch': ("The two password fields didn't match."),
            }
        new_password1 = forms.CharField(label=("New password"),
                                        widget=forms.PasswordInput)
        new_password2 = forms.CharField(label=("New password confirmation"),
                                        widget=forms.PasswordInput)

        def clean_new_password2(self):
            password1 = self.cleaned_data.get('new_password1')
            password2 = self.cleaned_data.get('new_password2')
            if password1 and password2:
                if password1 != password2:
                    raise forms.ValidationError(
                        self.error_messages['password_mismatch'],
                        code='password_mismatch',
                        )
            return password2


It will take two password input and verify if they match, if those inputs match(in clean method), it will return password. Now using that form, we are going to write a <b>view</b>(reference for implementation:(<a href="https://github.com/django/django/blob/731f313d604a6cc141f36d8a1ba9a75790c70154/django/contrib/auth/views.py#L192">source</a>)).


    class PasswordResetConfirmView(FormView):
        template_name = "account/test_template.html"
        success_url = '/admin/'
        form_class = SetPasswordForm

        def post(self, request, uidb64=None, token=None, *arg, **kwargs):
            """
            View that checks the hash in a password reset link and presents a
            form for entering a new password.
            """
            UserModel = get_user_model()
            form = self.form_class(request.POST)
            assert uidb64 is not None and token is not None  # checked by URLconf
            try:
                uid = urlsafe_base64_decode(uidb64)
                user = UserModel._default_manager.get(pk=uid)
            except (TypeError, ValueError, OverflowError, UserModel.DoesNotExist):
                user = None
        
            if user is not None and default_token_generator.check_token(user, token):
                if form.is_valid():
                    new_password= form.cleaned_data['new_password2']
                    user.set_password(new_password)
                    user.save()
                    messages.success(request, 'Password has been reset.')
                    return self.form_valid(form)
                else:
                    messages.error(request, 'Password reset has not been unsuccessful.')
                    return self.form_invalid(form)
            else:
                messages.error(request,'The reset password link is no longer valid.')
                return self.form_invalid(form)


URL for this view:


    urlpatterns += patterns('',
                           url(r'^admin/', include(admin.site.urls)),
                           url(r'^account/reset_password_confirm/(?P<uidb64>[0-9A-Za-z]+)-(?P<token>.+)/$', PasswordResetConfirmView.as_view(),name='reset_password_confirm'),
                       )



Well **PasswordResetConfirmView** takes two parameter from urls, uidb64 and token, those were sent within email generated by **ResetPasswordRequestView**. We got user id hence the user by decoding **uid64** by using **urlsafe__base64__decode**, and function **default___token___generator.check__token** checks the token against the user. If they are valid and the form is valid, we set new password for the user using **.set__password('password')** function. If they are not valid, it will show an error message saying the url is no longer valid.

More screenshots:(sequencial to implementation)
###Rendered template for SetPasswordForm form

<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/reset-password-prompt.png">

### Reset successful

<img src="https://dl.dropboxusercontent.com/u/31435562/django-reset-password/reset-success.png">

Thus you implement your very own forgot or reset password.

<h3><b>For full project/implementation, please check/fork this <a href="https://github.com/skyrudy/django-reset-password/tree/master">repository</a>. This code has been tested for Python3 and django 1.7/1.10 .</b> </h3>

Also don't forget to visit the original post at <a href="http://ruddra.com">my blog site</a>. Cheers!!

