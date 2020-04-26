# Simple Live Streaming Platform with Adaptive Bitrate

## Features

* Runs in AWS
* Optimized for low cost, capitalizes on the tiniest AWS virtual server instances
* Deploys in minutes
* RTMP broadcast
* HLS playback with Adaptive Bitrate
* Optional CDN delivery

## Setup

### Deploying the architecture

1. Sign in to the [AWS Management Console](https://aws.amazon.com/console), then click the button below to launch the CloudFormation template. Alternatively you can [download](template.yaml) the template and adjust it to your needs.

[![Launch Stack](https://cdn.jsdelivr.net/gh/buildkite/cloudformation-launch-stack-button-svg@master/launch-stack.svg)](https://console.aws.amazon.com/cloudformation/home#/stacks/create/review?stackName=live-streaming-hls-abr&templateURL=https://lostshadow.s3.amazonaws.com/abr-live-streaming-platform/template.yaml)

2. Choose your streaming server `InstanceType`:
	* `t3.nano` is the cheapest, and suitable for most scenarios
	* `t2.nano` is overall cheaper than `t3.nano` if you run your broadcast for less than ~30 minutes; after that it will slow down (due to depletion of launch credits) unless you enable [T2 Unlimited](https://aws.amazon.com/blogs/aws/new-t2-unlimited-going-beyond-the-burst-with-high-performance/)
	* `t2.micro` and `t3.micro` are similarly appropriate if you're taking advantage of the [AWS Free Tier](https://aws.amazon.com/free/), otherwise slightly more expensive than the nano
	* use the `t2.medium` just in case the other t2 variants do not work (unlikely), this being the cheapest t2-type dual-core instance
4. Set `UseCDN` to `yes` if you'd like to distribute your content via CDN; otherwise it will stream directly from the streaming server; the latter (`no`) is cheaper, however will only work up to about a hundred simultaneous viewers
5. Hit the `Create Stack` button. 
6. Wait for the `Status` to become `CREATE_COMPLETE`. Note that this may take up to **5 minutes** or more.
7. Under `Outputs`, notice the keys named `IngressUrl` and `PlaybackUrl/CDNPlaybackUrl`; write these down for using later

### Testing your setup

1. Set up your RTMP broadcaster (any of [these](https://support.google.com/youtube/answer/2907883) will work) at 720p (1280x720) and 2.5mbps

2. Point your RTMP broadcaster to the rtmp `IngressUrl` output by CloudFormation above

	Note that, while some RTMP broadcasters require a simple URI, others (like [OBS Studio](https://obsproject.com)) require a **Server** and **Stream key**. In this case, split the URI above at the last *slash* character, as following:
	
	**Server**: `rtmp://[HOST]/live`  
	**Stream key**: `stream001`

	Also note that **the enpoint may not be ready** as soon as CloudFormation stack is complete; it may take a couple minutes more for the server software to be compiled, installed and started on the virtual server so be patient

3. Test the video URL (`PlaybackUrl` or `CDNPlaybackUrl` output by CloudFormation above) in your favorite HLS player or player tester. You may use [this one](https://streaming4thepoor.live/index.php/hls-url-tester/) if not sure.

## Notes
* The "platform" is meant to support one stream at a time; if you need multiple simultaneous streams, the recommended approach is to launch the stack multiple times
* The ABR template features 3 transcoded bitrates (180p, 360p, 540p) and also reuses the original video (as a seprate quality variant) and audio(for all quality variants); the transcoding requirements are fine tuned to accomodate the CPU capabilities of a common .t2 or .t3 AWS instance; to modify the ABR settings you will need to alter the ffmpeg command (search for " -s " in the template) and also nginx's hls variant definitions (search for "hls_variant" in the template); any change should be tested to still be viable in the limited virtual server context, kindly mind it may just not work
