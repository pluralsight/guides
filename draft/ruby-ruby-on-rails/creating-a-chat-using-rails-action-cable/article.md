### THIS TUTORIAL IS IN DEVELOPMENT

-- intro

 Action Cable is a new feature of Rails that requires you to have the latest version. As of March 11, 2016, the latest stable version is Rails 5 beta3. In order to get it, you need to install the `rails` gem with the following options:

```bash
gem install rails --pre --no-ri --no-rdoc
```

Action Cable requires the [PostgreSQL](http://www.postgresql.org/download/) as its adapter. You need to install it on your Operating System and configure it before continuing with the tutorial.
When creating the a new Rails app, you need to specify it as an adapter so that the new app won't have the default SQLite.

```bash
rails _5.0.0.beta3_ new chatapp --database=postgresql
```

