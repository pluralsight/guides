CPU with multiple cores have become more and more common these days. To use full potential of the machine, writing multi threaded applications is now really important. Multi-threaded applications are difficult to code and test, expensive to maintain as they are prone to deadlock, race conditions and so many other multi-threaded hazards. 

Writing threaded application is not that tough these days as it used to be in the past. Dot net has already done so many heavy liftings for us so that we can focus more on application specifics. 

There are various ways of writing multi-threaded application in C#. 

## Starting Thread

    public class SimpleThreadExample
    {
        public void StartMultipleThread()
        {
            DateTime startTime = DateTime.Now;

            Thread t1 = new Thread(() =>
            {
                int numberOfSeconds = 0;
                while (numberOfSeconds < 5)
                {
                    Thread.Sleep(1000);

                    numberOfSeconds++;
                }

                Console.WriteLine("I ran for 5 seconds");
            });

            Thread t2 = new Thread(() =>
            {
                int numberOfSeconds = 0;
                while (numberOfSeconds < 8)
                {
                    Thread.Sleep(1000);

                    numberOfSeconds++;
                }

                Console.WriteLine("I ran for 8 seconds");
            });


            //parameterized thread
            Thread t3 = new Thread(p =>
            {
                int numberOfSeconds = 0;
                while (numberOfSeconds < Convert.ToInt32(p))
                {
                    Thread.Sleep(1000);

                    numberOfSeconds++;
                }

                Console.WriteLine("I ran for {0} seconds", numberOfSeconds);
            });

            t1.Start();
            t2.Start();
            //passing parameter to parameterized thread
            t3.Start(20);

            //wait for t1 to fimish
            t1.Join();

            //wait for t2 to finish
            t2.Join();

            //wait for t3 to finish
            t3.Join();


            Console.WriteLine("All Threads Exited in {0} secods", (DateTime.Now - startTime).TotalSeconds);
        }
        
## Terminating Thread

`Thread.Abort()` method can be used to destroy the running thread. But in practice it is better to let the CLR do the dirty work for you. You can have a flag which would break your long running process.

    public class DestroyThreadExample
    {
        public bool IsCancelled { get; set; }

        public Thread MyThread { get; set; }

        public void StartThread()
        {
            MyThread = new Thread(() =>
            {
                int numberOfSeconds = 0;
                while (numberOfSeconds < 8)
                {
                    if (IsCancelled == false)
                    {
                        break;
                    }

                    Thread.Sleep(1000);

                    numberOfSeconds++;
                }

                Console.WriteLine("I ran for {0} seconds", numberOfSeconds);
            });
        }

        //Destroy thread
        public void Abort()
        {
            MyThread.Abort();
        }

        //Graceful exit
        public void GracefulAbort()
        {
            IsCancelled = true;
        }
    }

## Responsive UI

One of the most popular use case of threading is to make the UI resposive. Lets see how we can achieve that with the simple thread and windows form.

    public partial class Form1 : Form
    {
        public delegate void UpdateLabel(string label);

        public bool IsCancelled { get; set; }

        public Form1()
        {
            InitializeComponent();
        }

        private void UpdateUI(string labelText)
        {
            lblStopWatch.Text = labelText;
        }

        private void btnStart_Click(object sender, EventArgs e)
        {
            DateTime startTime = DateTime.Now;

            IsCancelled = false;

            Thread t = new Thread(() =>
            {
                while (IsCancelled==false)
                {
                    Thread.Sleep(1000);

                    string timeElapsedInstring = (DateTime.Now - startTime).ToString(@"hh\:mm\:ss");

                    lblStopWatch.Invoke(new UpdateLabel(UpdateUI), timeElapsedInstring);
                    
                }
            });

          
            t.Start();
        }

        private void btnStop_Click(object sender, EventArgs e)
        {
            IsCancelled = true;
        }
    }

Noticed the delegate being used to update the UI? 

    lblStopWatch.Invoke(new UpdateLabel(UpdateUI), timeElapsedInstring);

This is because you can't update UI from any other thread other than the UI thread. I think this is very common scenario every one would have dealt with while working on windows forms.

There is another easiest way to update the UI.

## BackgroundWorker

There is another easiest way of creating a thread specifically to update the UI. 
BackgroundWorker

    public partial class Form2 : Form
    {
        BackgroundWorker workerThread = null;

        bool _keepRunning = false;

        public Form2()
        {
            InitializeComponent();

            InstantiateWorkerThread();
        }

        private void InstantiateWorkerThread()
        {
            workerThread = new BackgroundWorker();
            workerThread.ProgressChanged += WorkerThread_ProgressChanged;
            workerThread.DoWork += WorkerThread_DoWork;
            workerThread.RunWorkerCompleted += WorkerThread_RunWorkerCompleted;
            workerThread.WorkerReportsProgress = true;
            workerThread.WorkerSupportsCancellation = true;
        }

        private void WorkerThread_ProgressChanged(object sender, ProgressChangedEventArgs e)
        {
            lblStopWatch.Text = e.UserState.ToString();
        }

        private void WorkerThread_RunWorkerCompleted(object sender, RunWorkerCompletedEventArgs e)
        {
            if(e.Cancelled)
            {
                lblStopWatch.Text = "Cancelled";
            }
            else
            {
                lblStopWatch.Text = "Stopped";
            }            
        }

        private void WorkerThread_DoWork(object sender, DoWorkEventArgs e)
        {
            DateTime startTime = DateTime.Now;

            _keepRunning = true;

            while (_keepRunning)
            {
                Thread.Sleep(1000);
               
                string timeElapsedInstring = (DateTime.Now - startTime).ToString(@"hh\:mm\:ss");

                workerThread.ReportProgress(0, timeElapsedInstring);     
               
                if(workerThread.CancellationPending)
                {
                    // this is important as it set the cancelled property of RunWorkerCompletedEventArgs to true
                    e.Cancel = true;
                    break;
                }
            }
        }

        private void btnStart_Click(object sender, EventArgs e)
        {
            workerThread.RunWorkerAsync();
        }

        private void btnStop_Click(object sender, EventArgs e)
        {
            _keepRunning = false;
        }

        private void btnCancel_Click(object sender, EventArgs e)
        {
            workerThread.CancelAsync();          
        }
    }

In this approach, instead of creating the plain old thread and using delegate to update the UI, we have BackgroundWorker component which does the work for us. It supports multiple events to run long running process (DoWork), update the UI (ProgressChanged) and you will know when the background thread has actually ended (RunWorkerCompleted). In the plain old thread, knowing the end of the thread is tricky and you have to rely either of Thread.Join or use some other wait handles.

### Problems introduced by threading

While writing the multi-threaded application, there are a bunch of known issues that we should be able to handle. Deadlocks and race conditions are few to name. It is necessary to maintain synchronized access to different resources to make sure we are not corrupting the output. For example, if a file in the filesystem is being modified by multiple threads, the application must allow only one thread to modify the file at a time, otherwise the file might get corrupted. One way of doing it is using Lock keyword. If we are accessing the shared resource around the Lock statement, it will allow only one thread to execute the code within the lock block.

    public class LockExample
    {

        public int SharedResource { get; set; }

        public object _locker = 0;

        public void StartThreadAccessingSharedResource()
        {
            Thread t1 = new Thread(() =>
            {
                lock (_locker)
                {
                    SharedResource++;
                }

            });

            Thread t2 = new Thread(() =>
            {
                lock (_locker)
                {
                    SharedResource--;
                }

            });

            t1.Start();
            t2.Start();
        }
    }

In this example we have 2 threads accessing the same resource. Using the lock keyword will guarantee that the shared variable will be accessed by one one thread at a time. While thread t1 is executing the code within the lock block, thread t2 will be waiting.

These basics of c# multi-threading programming will prepare you for advance topics like Concurrent data structure, Wait handles, Tasks and asynchronous programming with C#.

associated code can be accessed @ https://github.com/bibekdw/CSharp-Threading-Tutorial-For-Beginners

