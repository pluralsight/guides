When users are saving or loading their states in applications or games, we simply read or write files from the disk to save or restore their states. However, sometimes you cannot use a single file to store information, because you may want to organize the current state into various groups, such as base text data, formatting, metadata etc.

In this tutorial I will explain how to generate and read files containing multiple files inside as a package.

### What are file packages?
Before we dive into the code I will give an example of a package file so we understand what we are trying to create. Various software use file packages to save and load states, such as Microsoft Office. If you played around or tried to restore an office file before, you know that you can open an Office file like this:
1. Change the Office file extension to zip.
2. Use your favorite compression utility to extract the file into a folder.
3. Open the folder

Here is an example.

We perform steps 1 and 2.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/66b37529-f446-46b4-95e4-0ba076cd9830.png)

Then we open the folder.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6d362a45-c483-4008-a7f1-cc87c0b1e544.png)

As you can see here, a Word file contains files and directories and it is actually acts as a wrapper of the files actually read and written by Office.

### Package Utility

First of all, our package utility will have 3 classes.

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

This class have two properties, one to hold the file path of the package and the second to hold the file paths of the contents of the file package. When we have these two fields ready we can start generating our package! Now let's create our `FilePackageWriter` class to actually write our package.

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

            _tempDirectoryPath = parentDirectoryPath + "\\" + filename + "_temp";
            if (Directory.Exists(_tempDirectoryPath))
            {
                Directory.Delete(_tempDirectoryPath, true);
            }

            Directory.CreateDirectory(_tempDirectoryPath);
            foreach (var filePath in _contentFilePathList)
            {
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

            ZipFile.CreateFromDirectory(_tempDirectoryPath, _filepath);
        }
        catch (Exception e)
        {
            var errorMessage = "An error occured while generating the package. " + e.Message;
            throw new Exception(errorMessage);
        }
        finally
        {
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

In this class we simply take the `FilePackage` and use the information to generate our package.
1. We find the parent directory path that we want to save the package into and the filename.
2. We create a temporary directory for our content files.
3. We copy the content files into this temporary directory.
4. We compress this directory and rename it as our package file.
5. 



