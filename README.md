# CollectLogsTest
This applcation collects logs and sends them

Collecting remote device logs without libraries or third party applications
Recently I’ve encountered a problem of remote debugging for Android applications. For exceptions I just use Firebase Crashlytics, but how do you collect logs from remote device if something is not working as expected without exceptions? In this guide I’ll show you how to collect logs with just standard Android/Java tools. You can get example app code from this repository. The process of collecting is devided in two steps:

Create Thread that will collect and save logs to files.
Send files to your developer account.
Creating a Thread.
Let’s create a class for log-collecting, I’m gonna call it LogcatHelper, and also we must create a Runnable for our Thread:


Let’s look at logcatRunnable, this Runnable will do most of the job. First, we create byte array buf, I’ve set it’s size to 10240 just to make sure my .log file won’t be too large, you can set whatever size you want. mLogInputStream is an InputStream that will read logs from logcatProc Process. Then, we create our .log file in cache folder with LogSaveHelper.prepareLogFile() static method, that file is going to be sent to your account:


Next, we create a Process object with the specified arguments:

private static final String[] PROC_ARRAY = new String[]{"logcat", "-v", "threadtime"};
With these arguments the Process will collect logs, and in order to get them we gonna use InputStream from that process:

mLogInputStream = logcatProc.getInputStream();
while loop simply reads data from process InputStream and writes it to file OutputStream which is returned by LogSaveHelper.getLogFileStream() method:


And that’s it for the first step.

Collecing and sending files
To collect files, we first need to stop while loop and close our InputStream/OutputStream. For that we just set saveSystemLog Boolean to false. And now we gonna grab the file from the cache folder and pack it in zip file.(Usually there is more than just one log file, so packing them in zip is good idea). Let’s use AsyncTask for that, I’ll name it CollectLogsTask:


In this example I’m going to construct applications info file, just to know what app version is installed by user, his OS version and so on (You should check github repository for more details). AppFilesHelper.constructAppInfoFile() will create a String containing app info, and AppFilesHelper.addAppInfoFile()
is going to create a File for that String. Next, we collect all files and put them in zip archive:


LogsFilenameFilter is a filter that will make sure we read only our log files.(You should get class code from repository as well). And the last step is to send the files. In this example Intent.ACTION_SEND_MULTIPLE is used to send files via gmail, but you can implement your own sending logic with different services( Like Telegram or WhatsApp ). We should specify recepient email, subject, title and to send our zip file we gonna create a Uri and attach it to intent with intent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, attachments):


onShowSendLogsDialog is a callback triggered from AsyncTask after files are collected, your activity/frament must implement CollectLogsTask.OnSendLogsDialogListener to receive that callback and make some UI changes. Don’t forget that you gonna need to create a FileProvider for cache folder in order to attach Uri, you can copy CachedFileProvider from repository.

Github repository: https://github.com/TheDanileron/CollectLogsTest
