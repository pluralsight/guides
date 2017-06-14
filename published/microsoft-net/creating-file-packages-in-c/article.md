Anytime users save or load something in an application or game, the program typically reads or writes files using the disk as a means of saving or restoring these "states." However, occasionally you cannot use a single file to store information. For instance, you may want to organize the current state into various groups, such as base text data, formatting, metadata, and so on. For that, the appropriate tool is a **file package.**

In this tutorial, I will explain how to generate and read packages that contain multiple files using C#. Hopefully, by the end of this guide, you will be able to understand the basics of file packages and be able to design customized packages, if need be.

## What are file packages?

Before we dive into the code I will give an example of a package file so we understand what we are trying to create. Various kinds of software use file packages to save and load states. One such example is Microsoft Office. If you played around or tried to restore an Office file before, you might have found that you can open a file like this:
1. Change the Office file extension to zip.
2. Use your favorite compression utility to extract the file into a folder.
3. Open the folder

We perform steps 1 and 2:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/66b37529-f446-46b4-95e4-0ba076cd9830.png)

Then we open the folder:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6d362a45-c483-4008-a7f1-cc87c0b1e544.png)

As you can see here, a Word file contains files and directories and it is actually acts as a wrapper of the files actually read and written by Office. This all forms a **file package**.

Now we know what file packages are, we can start writing the code for our packaging utility.

## Packaging Utility

Our packaging utility will have 3 classes.

1. A `FilePackage` class to hold information about our file packages.
2. A `FilePackageWriter` class to write file packages.
3. A `FilePackageReader` class to read file packages.

Let's get started by creating our first class.

```csharp
public class FilePackage
{
    public string FilePath { get; set; }
    public IEnumerable<string> ContentFilePathList { get; set; }
}
```

This class has two properties; `FilePath` to hold the file path of the package and `ContentFilePathList` to hold the file paths of the contents of the file package. Now let's create our `FilePackageWriter` class to actually write our package.

```csharp
public class FilePackageWriter
{
    private readonly string _filepath;
    private readonly IEnumerable<string> _contentFilePathList;
    private string _tempDirectoryPath;

    public FilePackageWriter(FilePackage filePackage)
    {
        _filepath = filePackage.FilePath;
        _contentFilePathList = filePackage.ContentFilePathList;
    }

    public void GeneratePackage(bool deleteContents)
    {
        try
        {
            string parentDirectoryPath = null;
            string filename = null;

            var fileInfo = new FileInfo(_filepath);

            // Get the parent directory path of the package file and if the package file already exists delete it
            if (fileInfo.Exists)
            {
                filename = fileInfo.Name;

                var parentDirectoryInfo = fileInfo.Directory;
                if (parentDirectoryInfo != null)
                {
                    parentDirectoryPath = parentDirectoryInfo.FullName;
                }
                else
                {
                    throw new NullReferenceException("Parent directory info was null!");
                }

                File.Delete(_filepath);
            }
            else
            {
                var lastIndexOfFileSeperator = _filepath.LastIndexOf("\\", StringComparison.Ordinal);
                if (lastIndexOfFileSeperator != -1)
                {
                    parentDirectoryPath = _filepath.Substring(0, lastIndexOfFileSeperator);
                    filename = _filepath.Substring(lastIndexOfFileSeperator + 1,_filepath.Length - (lastIndexOfFileSeperator + 1));
                }
                else
                {
                    throw new Exception("The input file path '" + _filepath +
                                        "' does not contain any file seperators.");
                }
            }

            // Create a temp directory for our package
            _tempDirectoryPath = parentDirectoryPath + "\\" + filename + "_temp";
            if (Directory.Exists(_tempDirectoryPath))
            {
                Directory.Delete(_tempDirectoryPath, true);
            }

            Directory.CreateDirectory(_tempDirectoryPath);
            foreach (var filePath in _contentFilePathList)
            {
                // Copy every content file into the temp directory we created before
                var filePathInfo = new FileInfo(filePath);
                if (filePathInfo.Exists)
                {
                    File.Copy(filePathInfo.FullName, _tempDirectoryPath + "\\" + filePathInfo.Name);
                }
                else
                {
                    throw new FileNotFoundException("File path " + filePath + " doesn't exist!");
                }
            }
            // Generate the ZIP from the temp directory
            ZipFile.CreateFromDirectory(_tempDirectoryPath, _filepath);
        }
        catch (Exception e)
        {
            var errorMessage = "An error occured while generating the package. " + e.Message;
            throw new Exception(errorMessage);
        }
        finally
        {
            // Clear the temp directory and the content files
            if (Directory.Exists(_tempDirectoryPath))
            {
                Directory.Delete(_tempDirectoryPath, true);
            }

            if (deleteContents)
            {
                foreach (var filePath in _contentFilePathList)
                {
                    if (File.Exists(filePath))
                    {
                        File.Delete(filePath);
                    }
                }
            }
        }
    }
}
```

In this class we simply take the `FilePackage` and use it to generate our package.
1. We find the parent directory path that we want to save the package into and the filename.
2. We create a temporary directory for our content files.
3. We copy the content files into this temporary directory.
4. We compress this directory and rename it as our package file.
5. Finally we delete the temporary folder we created, and if we want we can also delete the content files if we set the `deleteContents` parameter as `true`.

Now that we can write our packages, let's write the code to read them as well.
```csharp
public class FilePackageReader
{
    private Dictionary<string, string> _filenameFileContentDictionary;
    private readonly string _filepath;

    public FilePackageReader(string filepath)
    {
        _filepath = filepath;
    }

    public Dictionary<string, string> GetFilenameFileContentDictionary()
    {
        try
        {
            _filenameFileContentDictionary = new Dictionary<string, string>();

            // Open the package file
            using (var fs = new FileStream(_filepath, FileMode.Open))
            {
                // Open the package file as a ZIP
                using (var archive = new ZipArchive(fs))
                {
                    // Iterate through the content files and add them to a dictionary
                    foreach (var zipArchiveEntry in archive.Entries)
                    {
                        using (var stream = zipArchiveEntry.Open())
                        {
                            using (var zipSr = new StreamReader(stream))
                            {
                                _filenameFileContentDictionary.Add(zipArchiveEntry.Name, zipSr.ReadToEnd());
                            }
                        }
                    }
                }
            }

            return _filenameFileContentDictionary;
        }
        catch (Exception e)
        {
            var errorMessage = "Unable to open/read the package. " + e.Message;
            throw new Exception(errorMessage);
        }
    }
}
```

In the reader:
1. We get the file path of the package.
2. We open it using a `FileStream` and a `ZipArchive`.
3. We read the content files one by one and add the file name and the file contents to a `Dictionary`.

Our FilePackageWriter and FilePackageReader are finally ready. Now we can test our code and see if it works!
```csharp
var test1FilePath = AppDomain.CurrentDomain.BaseDirectory + "PackageTest\\test_1.txt";
var test2FilePath = AppDomain.CurrentDomain.BaseDirectory + "PackageTest\\test_2.txt";

using (var sw = new StreamWriter(test1FilePath))
{
    sw.WriteLine("test1");    
}

using (var sw = new StreamWriter(test2FilePath))
{
    sw.WriteLine("test2");
}

var packageFilePath = AppDomain.CurrentDomain.BaseDirectory + "PackageTest\\test.pkg";

var filePackage = new FilePackage
{
    FilePath = packageFilePath,
    ContentFilePathList = new List<string>
    {
        test1FilePath, test2FilePath
    }
};

var filePackageWriter = new FilePackageWriter(filePackage);
filePackageWriter.GeneratePackage(true);

var filePackageReader = new FilePackageReader(packageFilePath);
var filenameFileContentDictionary = filePackageReader.GetFilenameFileContentDictionary();

foreach (var keyValuePair in filenameFileContentDictionary)
{
    Console.WriteLine("Filename: " + keyValuePair.Key);
    Console.WriteLine("Content: " + keyValuePair.Value);
}
```

The output is:
```
Filename: test_1.txt
Content: test1

Filename: test_2.txt
Content: test2
```

We can also see it in the file system:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/14f3d633-f2b3-4c93-bd17-0190e34e9bbd.png)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7f91e4c2-cd7a-4550-b73c-faefff4746da.png)

This means that our packaging algorithms worked! 

## Conclusion

This guide explained how to generate file packages. Packaging is worthwhile when your application needs to read or write to multiple files when saving or restoring states. File packaging can eliminate unnecessary folders for each of your save files. Instead, your files stay neatly tucked in a single file that you can use to load or save states. This approach is also helpful to users because moving saved files into backup drives or sending them via e-mail becomes a streamlined process that no longer entails searching through folders.

I hope this guide will be useful for your projects. Let me know your ideas and feedback in the discussion section below. Happy coding!

