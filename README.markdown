# .NET Zencoder Wrapper #

A .NET wrapper for the Zencoder API written in C# using RestSharp.

### Why ###
Zencoder has a really nice API builder on their website and tools like John Sheehan's RestSharp make calling web services from .NET easier then ever. But why not take it a step further and get it down to one line of code to submit an encoding job? That's what this ZencoderWrapper was built to do.

### But there are already C# Wrappers for Zencoder ###

Yes! There are! And this one by Chad Burggraf is particularly excellent and you should check it out!
https://github.com/ChadBurggraf/zencoder-cs

I didn't find Chad's library until after writing my own. D'oh! No regrets, I learned some new things. 

I may end up using his library over my own for the asynchronous support. Part of it will depend on if Chad's library can compile with Mono since the personal project I want to build will ultimately be running on OSX. I'm not sure if my library will compile in Mono "as-is", but I believe RestSharp will and I don't think I used anything special beyond what's contained in RestSharp.

## Examples ##

One liner transcode:

    JobResponse job = new ZencoderClient(API_KEY).SubmitJob("http://cdeutsch.com/input.avi", "ftp://ftpuser:p4ssw0rd@cdeutsch.com/videos/", "output.mp4");


You'll need to create an account on Zencoder to use the service. And if you want to encode more then 5 seconds of video you'll need to pick a payment option. You can alternatively use the ZencoderWrapper to create your account:

    CreateAccountRequest account = new CreateAccountRequest("youremail@test.com", "password123");

    
Here's a more advanced example of creating an Ogg Vorbis file with thumbnails and notifications:

    ZencoderClient client = new ZencoderClient(API_KEY);
    JobRequest jobRequest = new JobRequest("http://cdeutsch.com/input.avi", new OutputSetting("ftp://ftpuser:p4ssw0rd@cdeutsch.com/videos/", "output.ogg"));
    //configure output settings.
    jobRequest.outputs[0].audio_codec = AudioCodecSetting.Vorbis;
    jobRequest.outputs[0].video_codec = VideoCodecSetting.Theora;
    //add a notification.
    jobRequest.outputs[0].NotificationSettings = new List<NotificationSetting>();
    jobRequest.outputs[0].NotificationSettings.Add(new NotificationSetting(NotificationType.Email, "youremail@test.com"));
    //create thumbnails
    jobRequest.outputs[0].thumbnails = new ThumbnailSetting();
    jobRequest.outputs[0].thumbnails.base_url = "ftp://ftpuser:p4ssw0rd@cdeutsch.com/thumbs/";
    jobRequest.outputs[0].thumbnails.format = ThumbnailFormatSetting.PNG;
    jobRequest.outputs[0].thumbnails.interval = 5;
    jobRequest.outputs[0].thumbnails.number = 3;
    jobRequest.outputs[0].thumbnails.prefix = "thumb_";
    jobRequest.outputs[0].thumbnails.size = "120x80";
    jobRequest.outputs[0].thumbnails.start_at_first_frame = true;
    //submit the job
    JobResponse job = client.SubmitJob(jobRequest);

    
Here's how to check the status of the job we just created:

    //get job details.
    JobListingResponse job = client.GetJob(job.id);

    
Here's how to check to the status of the job's output file (there can be multiple outputs):

    //get progress of first (in this case only) output file.
    JobOutputProgressResponse progress = client.GetJobOutputProgress(job.outputs[0].id);

    
And finally here's how to get a list of all of your jobs:

    //get list of jobs  
    List<JobListingResponse> jobList = client.ListJobs();  
	
## History ##

### 6/15/2011 ###
* Merge from craigkitterman: Added support for S3 bucket HTTP headers in OutputSettings.

### 6/02/2011 ###
* Fixed bug with frame rate needing to be a decimal.
* Updated Thumbmail class to support new return values.
* Updated to rethrow RestSharp errors.