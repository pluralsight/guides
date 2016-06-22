Last year I needed to build a **W**indows **P**resentation **F**oundation (WPF) application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, ofcourse, very *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe, to learn something properly, you need to create a basic project using it. So in this tutorial I will explain how to build a simple media player from ground up using a very popular .NET audio library NAudio.

### Why Build Your Own Media Player?
Other than the fact that it is fun:
- You might have a project where you need to make a in-app media player to achieve your goals like my situation.
- You might need a very special feature that no other media player provides.
- You might also not trust 3rd party media players in your secure environment where you absolutely have to use a media player.

Whatever your reason is, you have challenges waiting for you down the road when it comes to audio programming.

### Challenges
The first challenge is the concept that audio is not a traditional object, but a stream object. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it can't be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achive what you want. They are not very easy to debug either. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc. Especially when it comes to debugging it can be a mess.

There are questions need answering such as "How do we know if we reach the end of an audio file?", "How do we stop or pause the stream?", "How do we resume where we left off?". Ofcourse there are many more issues you need to address but these are the core issues.

Finally there are memory management issues. Typically, when you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundereds of audio and you don't dispose them properly, you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

So when you think about audio considering these challenges, even a simple action such as playing an audio file, gets very complicated very fast.

### NAudio to the Rescue
Lucky for us, there is an audio library for .NET - NAudio - that will do most of this work for us. We will still be working with streams and we will have to face several challanges that comes with them, however NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio libarary is, it can be used in every type of project. If you simply want build and application to play or record audio files, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters it provides very good tools for that too.

There are 3 ways of getting NAudio:
- [NAudio codeplex page](https://naudio.codeplex.com/)
- [NAudio GitHub page](https://github.com/naudio/NAudio)
- And finally via [NuGet](https://www.nuget.org/packages/NAudio/)

I usually go with NuGet since it is the easiest of them all, however if you want to see how it actually works there are great examples and documentation on the Codeplex and GitHub pages.

### Media Player

The main purpose of this tutorial is to build a simple media player which will have some common features of various media players such as:
- Play audio files with various formats (wav, mp3, ogg, flac etc)
- Skip to beginning, skip to end, play/pause, stop, shuffle buttons and their functionality
- Have a volume control
- Have a seekbar
- Have a playlist
- Add a file to this playlist
- Add a folder of files to this playlist
- Save and load playlists
- When a track finishes, it will move to the next track in the playlist automatically.

It should look like this at the end (with my Witcher 3 soundtrack playlist):
![Our finalized media player](https://raw.githubusercontent.com/pluralsight/guides/master/images/71e4ccf5-4301-4b3f-8cb2-7fde9b7a816e.png)

### Implementation
In the solution, we will have two projects. One for the actual application, and one for the NAudio abstraction. NAudio is great in abstracting us from details but I want to make it very easy for us to do everything with a few methods, rather than calling NAudio codes for playing, pausing, stopping etc.

However there is a choice we have to make when it comes to the main project where the UI is; do we use the traditional event driven architecture or do we use **M**odel **V**iew **V**iew **M**odel architecture (MVVM). You might think that you would use MVVM because that's all the cool kids use nowadays, however both architectures have advantages and disadvantages especially when it comes to developing a real time application such as this. I developed media players using both architectures on two different projects:
- If you use MVVM you write way less code, however in this case, property binding has a nasty habit of breaking down and causing bugs if you bind the wrong way. Especially when it comes to implementing a real time changing two-way bound seekbar control.
- If you use the good old event driven architecture you get to write a lot more **U**ser **I**nterface (UI) event code, however you will have full control over what happens during those events so it is easier to implement a real time control such as a seekbar.

Since this is a brand new project I would go with MVVM, so in this tutorial I will use the MVVM architecture. However since this is not an MVVM tutorial I won't go into details on how MVVM works.

So now that we choose our architecture we need to generate the right namespaces in our project. Our solution structure will be like this:
- Solution
  - Project: NaudioPlayer
    - Models
    - ViewModels
    - Views
    - Services
    - Images
  - Project: NaudioWrapper

#### Mocking up the UI
I always like to start with the UI part when it comes to WPF projects because it provides me with a feature list I need to implement in a visual way. So I will start coding by first generating a UI mock-up so we can see what features we need to code.

Before we start on the UI however, I will provide you the link for the images I used for the buttons so you will have them ready. For icons I used Google's free material design icons. You can get them from Google's material design [page](https://material.google.com/resources/sticker-sheets-icons.html#). When you download the icons, do a quick search of "play" or "pause" in the download directory to find the relevant icons.

We also need `System.Windows.Interactivity` and `Microsoft.Expression.Interaction` namespaces for our event bindings. I said we won't use event driven architecture but sometimes there is no way avoiding events. So these namespaces provide events the MVVM way. Adding these namespaces can be tricky and might not always work as you expect them to work. These namespaces actually come with Blend, a UI development tool for XAML based projects. So if you have Blend installed (it comes with the Visual Studio installer), you can find those DLL files from:

- For .NET 4.5+ `C:\Program Files (x86)\Microsoft SDKs\Expression\Blend\.NETFramework\v4.5\Libraries`
- For .NET 4.0 `C:\Program Files (x86)\Microsoft SDKs\Expression\Blend\.NETFramework\v4.0\Libraries`

If you don't have Blend installed just run the Visual Studio installer and modify/add it to your installation.

However, in my experience adding those in Visual Studio doesn't always work. So what I usually do:
- Create the WPF project in Visual Studio
- Add `xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"` to the `Window`.
- Close Visual Studio and open the project in Blend.
- From the menu click Project->Add Reference and add the references from that menu.
- Close Blend and open the project back in Visual Studio

Now everything ready for our UI code.

Here is our E**x**tensible **A**pplication **M**arkup **L**anguage (XAML) code for the UI:
```xaml
<Window x:Class="NaudioPlayer.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:NaudioPlayer"
        xmlns:i="http://schemas.microsoft.com/expression/2010/interactivity"
        xmlns:viewModels="clr-namespace:NaudioPlayer.ViewModels"
        mc:Ignorable="d"
        Title="{Binding Title}" Height="350" Width="525">
    <Window.DataContext>
        <viewModels:MainWindowViewModel/>
    </Window.DataContext>
    <DockPanel LastChildFill="True">
        <Menu DockPanel.Dock="Top">
            <MenuItem Header="File">
                <MenuItem Header="Save Playlist" Command="{Binding SavePlaylistCommand}"/>
                <MenuItem Header="Load Playlist" Command="{Binding LoadPlaylistCommand}"/>
                <MenuItem Header="Exit" Command="{Binding ExitApplicationCommand}"/>
            </MenuItem>
            <MenuItem Header="Media">
                <MenuItem Header="Add File to Playlist..." Command="{Binding AddFileToPlaylistCommand}"/>
                <MenuItem Header="Add Folder to Playlist..." Command="{Binding AddFolderToPlaylistCommand}"/>
            </MenuItem>
        </Menu>
        <Grid DockPanel.Dock="Bottom" Height="30">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="30"/>
                <ColumnDefinition Width="*"/>
                <ColumnDefinition Width="30"/>
            </Grid.ColumnDefinitions>
            <Button Grid.Column="0" Margin="3" Command="{Binding RewindToStartCommand}">
                <Image Source="../Images/skip_previous.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="1" Margin="3" Command="{Binding StartPlaybackCommand}">
                <Image Source="{Binding PlayPauseImageSource}" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="2" Margin="3" Command="{Binding StopPlaybackCommand}">
                <Image Source="../Images/stop.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="3" Margin="3" Command="{Binding ForwardToEndCommand}">
                <Image Source="../Images/skip_next.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
            <Button Grid.Column="5" Margin="3" Command="{Binding ShuffleCommand}">
                <Image Source="../Images/shuffle.png" VerticalAlignment="Center" HorizontalAlignment="Center"/>
            </Button>
        </Grid>
        <Grid DockPanel.Dock="Bottom" Margin="3">
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="3*"/>
                <ColumnDefinition Width="20"/>
                <ColumnDefinition Width="20"/>
                <ColumnDefinition Width="*"/>
            </Grid.ColumnDefinitions>
            <Slider Grid.Column="0" Minimum="0" Maximum="{Binding CurrentTrackLenght, Mode=OneWay}" Value="{Binding CurrentTrackPosition, Mode=TwoWay}" x:Name="SeekbarControl" VerticalAlignment="Center">
                <i:Interaction.Triggers>
                    <i:EventTrigger EventName="PreviewMouseDown">
                        <i:InvokeCommandAction Command="{Binding TrackControlMouseDownCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                    <i:EventTrigger EventName="PreviewMouseUp">
                        <i:InvokeCommandAction Command="{Binding TrackControlMouseUpCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                </i:Interaction.Triggers>
            </Slider>
            <Image Grid.Column="2" Source="../Images/volume.png"></Image>
            <Slider Grid.Column="3" Minimum="0" Maximum="1" Value="{Binding CurrentVolume, Mode=TwoWay}" x:Name="VolumeControl" VerticalAlignment="Center">
                <i:Interaction.Triggers>
                    <i:EventTrigger EventName="ValueChanged">
                        <i:InvokeCommandAction Command="{Binding VolumeControlValueChangedCommand}"></i:InvokeCommandAction>
                    </i:EventTrigger>
                </i:Interaction.Triggers>
            </Slider>
        </Grid>
        <ListView x:Name="Playlist" ItemsSource="{Binding Playlist}" SelectedItem="{Binding CurrentTrack, Mode=TwoWay}">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition></ColumnDefinition>
                        </Grid.ColumnDefinitions>
                        <TextBlock Grid.Column="0" Text="{Binding Path=FriendlyName, Mode=OneWay}"></TextBlock>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </DockPanel>
</Window>
```

So by looking at our XAML code, in our ``MainWindowViewModel`` viewmodel, we need to implement the following commands

- Menu Commands
  - **SavePlaylistCommand:** This command will save our `Playlist` into a text file, simply by writing the path of each `Track` in the `Playlist` as `<filename>.playlist`.
  - **LoadPlaylistCommand:** This command will read the `.playlist` file of our choosing and generate a `Playlist` for us.
  - **ExitApplicationCommand:** This command will dispose all our audio streams and close the application.
  - **AddFileToPlaylistCommand:** This command will add a single audio file to our `Playlist`.
  - **AddFolderToPlaylistCommand:** This command will add a folder of files to our `Playlist`.
- Player Commands
  - **RewindToStartCommand:** This command will set the `CurrentTrackPosition` to 0, effectively skipping to start.
  - **StartPlaybackCommand:** This command will start playback of the `CurrentTrack`. When pressed during playback, it will pause the playback.
  - **StopPlaybackCommand:** This command will stop the playback.
  - **ForwardToEndCommand:** This command will skip to the last second of the currently playing audio file.
  - **ShuffleCommand:** This command will shuffle our `Playlist` randomly.
- MVVM Events
  - **TrackControlMouseDownCommand:** This command will run when we press mouse button on the seekbar slider control.
  - **TrackControlMouseUpCommand:** This command will run when we release mouse button from the seekbar slider control.
  - **VolumeControlValueChangedCommand:** This command will run when we change the value of the volume control slider.

and properties:
- **Title:** Title of the window.
- **CurrentTrackPosition:** While the audio is playing, this property is used to store the current position in seconds.
- **CurrentVolume:** This property sets the volume.
- **Playlist:** This property represents our playlist.
- **CurrentTrack:** This property represents currently selected track in our playlist.
- **PlayPauseImageSource:** This property sets either play or pause images depending on whether the audio is playing or paused to our play button.

### NaudioWrapper
Before we move on to the ViewModel we need the basic features abstracted away from the core NAudio code. To do this I will create an `AudioPlayer` class to hold all these features in its methods and our ViewModel can access these public methods to interact with it.

Looking at our feature list, 
```cs
public class AudioPlayer
{

}
```


### The ViewModel
Before we implement the ViewModel we need to define how commands work. We will use the commonly used `RelayCommand` class to do this:
```cs
public class RelayCommand : ICommand
{
    private readonly Action<object> _execute;
    private readonly Predicate<object> _canExecute;

    public RelayCommand(Action<object> execute, Predicate<object> canExecute)
    {
        _execute = execute;
        _canExecute = canExecute;
    }
    public bool CanExecute(object parameter)
    {
        bool result = _canExecute == null ? true : _canExecute(parameter);
        return result;
    }

    public void Execute(object parameter)
    {
        _execute(parameter);
    }

    public event EventHandler CanExecuteChanged
    {
        add { CommandManager.RequerySuggested += value; }
        remove { CommandManager.RequerySuggested -= value; }
    }
}
```

Now, let's stub out our ViewModel:

```cs
public class MainWindowViewModel : INotifyPropertyChanged
{
    private string _title;
    private double _currentTrackLenght;
    private double _currentTrackPosition;
    private string _playPauseImageSource;
    private float _currentVolume;

    private Track _currentTrack;        
    private ObservableCollection<Track> _playlist;        

    public string Title
    {
        get { return _title; }
        set
        {
            if (value == _title) return;
            _title = value;
            OnPropertyChanged(nameof(Title));
        }
    }

    public string PlayPauseImageSource
    {
        get { return _playPauseImageSource; }
        set
        {
            if (value == _playPauseImageSource) return;
            _playPauseImageSource = value;
            OnPropertyChanged(nameof(PlayPauseImageSource));
        }
    }

    public float CurrentVolume
    {
        get { return _currentVolume; }
        set
        {

            if (value.Equals(_currentVolume)) return;
            _currentVolume = value;
            OnPropertyChanged(nameof(CurrentVolume));
        }
    }

    public double CurrentTrackLenght
    {
        get { return _currentTrackLenght; }
        set
        {
            if (value.Equals(_currentTrackLenght)) return;
            _currentTrackLenght = value;
            OnPropertyChanged(nameof(CurrentTrackLenght));
        }
    }

    public double CurrentTrackPosition
    {
        get { return _currentTrackPosition; }
        set
        {
            if (value.Equals(_currentTrackPosition)) return;
            _currentTrackPosition = value;
            OnPropertyChanged(nameof(CurrentTrackPosition));
        }
    }

    public Track CurrentTrack
    {
        get { return _currentTrack; }
        set
        {
            if (Equals(value, _currentTrack)) return;
            _currentTrack = value;
            OnPropertyChanged(nameof(CurrentTrack));
        }
    }        

    public ObservableCollection<Track> Playlist
    {
        get { return _playlist; }
        set
        {
            if (Equals(value, _playlist)) return;
            _playlist = value;
            OnPropertyChanged(nameof(Playlist));
        }
    }

    public ICommand ExitApplicationCommand { get; set; }
    public ICommand AddFileToPlaylistCommand { get; set; }
    public ICommand AddFolderToPlaylistCommand { get; set; }
    public ICommand SavePlaylistCommand { get; set; }
    public ICommand LoadPlaylistCommand { get; set; }

    public ICommand RewindToStartCommand { get; set; }
    public ICommand StartPlaybackCommand { get; set; }
    public ICommand StopPlaybackCommand { get; set; }
    public ICommand ForwardToEndCommand { get; set; }
    public ICommand ShuffleCommand { get; set; }

    public ICommand TrackControlMouseDownCommand { get; set; }
    public ICommand TrackControlMouseUpCommand { get; set; }
    public ICommand VolumeControlValueChangedCommand { get; set; }

    public event PropertyChangedEventHandler PropertyChanged;

    public MainWindowViewModel()
    {		
        LoadCommands();
		
        Playlist = new ObservableCollection<Track>();            
		
		Title = "NaudioPlayer";
        PlayPauseImageSource = "../Images/play.png";            
    }       

    private void LoadCommands()
    {
        // Menu commands
        ExitApplicationCommand = new RelayCommand(ExitApplication,CanExitApplication);
        AddFileToPlaylistCommand = new RelayCommand(AddFileToPlaylist, CanAddFileToPlaylist);
        AddFolderToPlaylistCommand = new RelayCommand(AddFolderToPlaylist, CanAddFolderToPlaylist);
        SavePlaylistCommand = new RelayCommand(SavePlaylist, CanSavePlaylist);
        LoadPlaylistCommand = new RelayCommand(LoadPlaylist, CanLoadPlaylist);

        // Player commands
        RewindToStartCommand = new RelayCommand(RewindToStart, CanRewindToStart);
        StartPlaybackCommand = new RelayCommand(StartPlayback, CanStartPlayback);
        StopPlaybackCommand = new RelayCommand(StopPlayback, CanStopPlayback);
        ForwardToEndCommand = new RelayCommand(ForwardToEnd, CanForwardToEnd);
        ShuffleCommand = new RelayCommand(Shuffle, CanShuffle);

        // Event commands
        TrackControlMouseDownCommand = new RelayCommand(TrackControlMouseDown, CanTrackControlMouseDown);
        TrackControlMouseUpCommand = new RelayCommand(TrackControlMouseUp, CanTrackControlMouseUp);
        VolumeControlValueChangedCommand = new RelayCommand(VolumeControlValueChanged, CanVolumeControlValueChanged);
    }

    // Menu commands
    private void ExitApplication(object p)
    {
        
    }
    private bool CanExitApplication(object p)
    {
        return true;
    }

    private void AddFileToPlaylist(object p)
    {
        
    }
    private bool CanAddFileToPlaylist(object p)
    {
        return true;
    }

    private void AddFolderToPlaylist(object p)
    {
        
    }

    private bool CanAddFolderToPlaylist(object p)
    {
        return true;
    }

    private void SavePlaylist(object p)
    {
        
    }

    private bool CanSavePlaylist(object p)
    {
        return true;
    }

    private void LoadPlaylist(object p)
    {
        
    }

    private bool CanLoadPlaylist(object p)
    {
        return true;
    }

    // Player commands
    private void RewindToStart(object p)
    {
        
    }
    private bool CanRewindToStart(object p)
    {
        return true;
    }

    private void StartPlayback(object p)
    {
        
    }

    private bool CanStartPlayback(object p)
    {
        return true;
    }

    private void StopPlayback(object p)
    {
        
    }
    private bool CanStopPlayback(object p)
    {
        return true;
    }

    private void ForwardToEnd(object p)
    {
        
    }
    private bool CanForwardToEnd(object p)
    {
        return true;
    }

    private void Shuffle(object p)
    {
        
    }
    private bool CanShuffle(object p)
    {
        return true;
    }

    // Events
    private void TrackControlMouseDown(object p)
    {
        
    }

    private void TrackControlMouseUp(object p)
    {
        
    }

    private bool CanTrackControlMouseDown(object p)
    {
        return true;
    }

    private bool CanTrackControlMouseUp(object p)
    {
        return true;
    }

    private void VolumeControlValueChanged(object p)
    {
        
    }

    private bool CanVolumeControlValueChanged(object p)
    {
        return true;
    }

    [NotifyPropertyChangedInvocator]
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```











