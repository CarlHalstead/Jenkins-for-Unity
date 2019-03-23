# Jenkins for Unity

## What is Jenkins?

From the Jenkins website:

```
The leading open source automation server, Jenkins provides hundreds of plugins to support building, 
deploying and automating any project.
```

What this allows us to do is take a Unity project currently on GitHub (Or BitBucket, GitLab etc) and automatically build the project to the platforms specified. Once the build is complete we can then zip the build and automatically transfer it to a Google Drive folder for upload. This allows you to build all of your target platforms with no user intervention and without the building processing blocking your Unity instance so you can continue working.

## Prerequisites

1. [Jenkins](https://jenkins.io) (This should be installed on another machine and you should be able to sign in. This is done through the web browser at LocalIpOfJenkinsMachine:8080. You can find the IP address of the Jenkins machine by using the command 'ipconfig' in Command Prompt on said machine)
2. [Google Drive](https://www.google.com/drive/) / [Dropbox](https://www.dropbox.com) / Other Cloud Storage (This is used to store your builds. This allows you and others access to the builds from anywhere and allows you to share them with other extremely easily)
3. [This Git repository](https://github.com/GamingAnonymous/Jenkins-for-Unity) imported into your Unity project. The vital files include:
    1. The Jenkins folder. This contains the scripts needed to build the application.
    2. The csc and mcs files. These import the required DLLs to zip your build. They MUST be in your root assets folder otherwise the ZipFile class will not be recognised.
4. [Git](https://git-scm.com) installed on your Jenkins machine. This will be used to clone your Git repository.

## Getting Started (Jenkins Setup)

Before starting with Jenkins for Unity you need to take care of a few things within Jenkins.

1. Once setup and signed in, you are going to want to go to 'Manage Jenkins' -> 'Manage Plugins' -> 'Installed' and check if the following plugins are installed:
   * 'Git plugin'
   * 'GitHub Integration Plugin'
   * 'Github Branch Source Plugin'
   
   ![Installed plugins](https://i.imgur.com/N5GjB8c.png)

    If any of these are not installed go to the 'Available' tag, search for any that are missing and install them.

    I also recommend getting ensuring that you have the 'Build Timeout' plugin. This will allow you to set a time to cancel a build       if it gets stuck in the build process somehow.
    
2. Next, go to 'Manage Jenkins' -> 'Global Tool Configuration' and go to 'Git' and click 'Add Git' -> 'Git' and paste the path for Git.exe which you should have installed you your machine. Mine is located at: 'C:\Program Files\Git\bin\git.exe'. Give your Git install a name, I call mine 'Default'.

![Git installation](https://i.imgur.com/Q1SpuWI.png)
    
3. You can now return to the Jenkins homepage and select 'New Item' in the top left hand corner of the page. Give your project a suitable name. (I recommend the same name as your Unity project for simplicity). Then you can select 'Freestyle project' and hit 'OK'.

![New Jenkins Item](https://i.imgur.com/HYjV4Gl.png)

4. Now you will be greeted with a page containing a lot of options. This is fine and can be daunting at first. Lets break it down.
    1. You will want to tick 'GitHub project' (Assuming your project is on GitHub) and paste a link to your project. It should follow the pattern http://github.com/AccountName/ProjectName
    
    ![GitHub Project URL](https://i.imgur.com/nTSBOzV.png)
    
    2. Move down to the 'Source Code Management' section and select 'Git'.
    3. Input the 'Repository URL'. This will be the same as your 'Github project' URL entered previously but with '.git' at the end.     This is the URL that Git will clone from.
    
    ![Git Repository](https://i.imgur.com/dbmzsTC.png)
    
    4. Now, time to add your GitHub credentials. Do not worry, you do not need your password for this!
        1. Go to [https://github.com/settings/tokens](https://github.com/settings/tokens)
        2. Select 'Generate new token'.
        3. Give your token a friendly but descriptive name 'Carl-Laptop=Jenkins' for example.
        4. Tick the box which says 'repo'. This will allow Jenkins to build any of your projects, public or private.
        5. Click 'Generate token' and after returning to the settings page, remember to copy the token! (You will get an email from         GitHub telling you that a new token has been created)
        
        ![GitHub Token](https://i.imgur.com/44TTklI.png)

    5. Return to Jenkins and click the 'Add' -> 'Jenkins' button next to 'Credentials' to add your login. The username, is your normal GitHub username. The password used here, will be the token you generated just a moment ago. Click 'Add'.
    
    ![GitHub Credentials](https://i.imgur.com/GJOkNSg.png)
    
    6. If you would like to build a specific branch then type it into the 'Branches to build' input box. Personally, I will be building the master branch.
    
    ![Branch Selection](https://i.imgur.com/TfRIRRR.png)
    
    7. Onto the 'Build Triggers' section. Tick 'Poll SCM' and type "H/5 * * * *" into the box (Minus the quotes). This means that it will 'poll' (or check) your Git repository for any changes every 5 minutes. You can change the 5 to any number of minutes you desire. If you would like builds sooner, by all means, set it to 1 minute.
    
    ![GitHub Polling](https://i.imgur.com/fhXg9Ri.png)
    
    8. Onto the 'Build Environment' section. The only box I tick here is 'Add timestamps to the Console Output'. This is personal preference and I just use it help track how long things take. It is not needed however.
    
    ![Timestamps](https://i.imgur.com/f5HSjc5.png)
    
    9. Onto the 'Build' section. This is the meat of the configuration. This is where you will open Unity and Invoke the plugin to build your project. Here, I have a premade script (See bottom of this Getting Started section) that works for me and may act as a template for you to use. My script assumes that you are using Unity Hub. If you are not, I recommend it and will simplify this process. Now, I suggest copying my Batch file and making the following edits:
        1. Change the UnityHubLocation variable below to your own Unity Hub path. It should follow like so: <HubInstallLocation>/Unity/Hub
        2. Change the UnityVersion variable to the specific version that you would like to build your project with. This should already be installed in the Unity Hub and the name of the version should be be exactly how it is shown within the Hub. 
    
     ![Jenkins Batch File](https://i.imgur.com/QPHSwg2.png)
  
    10. Finally, in the 'Post-build Actions' section, click 'Add post-build action' and click 'Archive the artifacts'. Then enter the following into the input box: 'Logs\UnityEditor.log' (Minus the quotes). What this will do, is upload the editor log to Jenkins so it can be viewed remotely. This will allow you to see why any builds have failed so you can rectify it.
    
    ![Jenkins Artifacts](https://i.imgur.com/2BcGsiV.png)
  
That is about all for the Jenkins part of setting up your project. The next part will be the Unity end.

### Jenkins Batch file:

```
set UnityHubLocation=C:\Program Files\Unity\Hub
set UnityVersion=2018.3.8f1

echo Attempting to build %JOB_NAME% in Unity Version %UnityVersion%.
echo Workspace location: '%WORKSPACE%''
echo Unity Hub location: '%UnityHubLocation%''

"%UnityHubLocation%\Editor\%UnityVersion%\Editor\Unity.exe" -projectPath "%WORKSPACE%" -quit -nographics -batchmode -logFile "%WORKSPACE%\Logs\UnityEditor.log" -executeMethod UnityBuild.BuildPlatforms

echo Build finished, see attached log for information!
```

## Getting Started (Unity Setup)

Now that your Jenkins is all set-up it is time to move onto the Unity section. You should have already imported this repository into your project. (Can be done by using the green download button on GitHub and dragging the Jenkins folder, mcs and csc files into your project).

1. Within Unity open the file 'Jenkins' -> 'Editor' -> 'UnityBuild.cs'.
2. Replace my path to the Android SDK (AndroidSdkDirectory) with your own. (Line 28).
3. Replace my path to the temp build directory (DriveTempDirectory) with your own. (Line 97).
4. Replace my path to the final build directory (DriveDirectory) with your own (Line 100).

Each of these paths is explained within the code.

### Selecting which platforms to build.

By default, it is setup to build for all current platforms that the code supports. This includes Windows/Mac/Linux, Android, iOS and WebGL. You can disable the build for each platform like so:

```
private static readonly Dictionary<BuildTarget, PlatformBuilds> PlatformToBuild = new Dictionary<BuildTarget, PlatformBuilds>()
{
  { BuildTarget.StandaloneWindows64,      new PlatformBuilds(BuildWindows, true) },
  { BuildTarget.StandaloneLinuxUniversal, new PlatformBuilds(BuildLinux,   true) },
  { BuildTarget.StandaloneOSX,            new PlatformBuilds(BuildMacOS,   true) },
  { BuildTarget.Android,                  new PlatformBuilds(BuildAndroid, true) },
  { BuildTarget.iOS,                      new PlatformBuilds(BuildiOS,     true) },
  { BuildTarget.WebGL,                    new PlatformBuilds(BuildWebGL,   true) },
};
```

Where the boolean is whether or not Jenkins will build for that platform. For example, if you only wanted to build for Android, you would change it to:

```
private static readonly Dictionary<BuildTarget, PlatformBuilds> PlatformToBuild = new Dictionary<BuildTarget, PlatformBuilds>()
{
  { BuildTarget.StandaloneWindows64,      new PlatformBuilds(BuildWindows, false) },
  { BuildTarget.StandaloneLinuxUniversal, new PlatformBuilds(BuildLinux,   false) },
  { BuildTarget.StandaloneOSX,            new PlatformBuilds(BuildMacOS,   false) },
  { BuildTarget.Android,                  new PlatformBuilds(BuildAndroid, true)  },
  { BuildTarget.iOS,                      new PlatformBuilds(BuildiOS,     false) },
  { BuildTarget.WebGL,                    new PlatformBuilds(BuildWebGL,   false) },
};
```

Nice and simple. Do not forget to commit this change however otherwise Jenkins will not be able to see the changes!

## Now you are done, you can press 'Build Now' in Jenkins if you so wish! Feedback is definitely appreciated.
