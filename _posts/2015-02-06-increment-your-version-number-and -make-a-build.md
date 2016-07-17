---
layout: post
title: Increment Your Version Number and Make a Build 
---


I am a fan of [semantic versioning][SV]. And in all of the projects here at [Ice House][ICE] we enforce to do it so that we can make sure which build is the right build to be tested and shipped to the App Store.

It is kind of tedious things to just increment your version every time your fixes or updates are ready to be tested.

So I look up some scripts on the giant Internet and found [this][SEK]. 
The problem is, the script is written to be run by placing it as Run Script on the Build Phase. Since we use xcodebuild and [Shenzhen][SHZ] to automate things we need the script to be run along other scripts.

So I modified it a little:

```bash
APP_NAME="YourAppName"
INFOPLIST_FILE="${PWD}/${APP_NAME}/Supporting Files/Info.plist"

VERSIONNUM=$(/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" "${INFOPLIST_FILE}")
NEWSUBVERSION=`echo $VERSIONNUM | awk -F "." '{print $3}'`
NEWSUBVERSION=$(($NEWSUBVERSION + 1))
NEWVERSIONSTRING=`echo $VERSIONNUM | awk -F "." '{print $1 "." $2 ".'$NEWSUBVERSION'" }'`
/usr/libexec/PlistBuddy -c "Set :CFBundleShortVersionString $NEWVERSIONSTRING" "${INFOPLIST_FILE}"

echo "Set version to $NEWVERSIONSTRING"

```

Then I save it in my root project directory and run it before making an ipa build.

Notice that it will increment the last digit on the build. From here, you can modify the script above to update all plist files which contain bundle version on your other targets in your project.

Then from here, you can make build with Shenzhen:

	ipa build --verbose
	
and push it to itunesconnect 
	
	ipa distribute:itunesconnect

or [other][DIS] build distribution services supported.


[SV]:http://semver.org
[ICE]:http://icehousecorp.com
[SEK]:https://gist.github.com/sekati/3172554#file-xcode-version-bump-sh
[SHZ]:https://github.com/nomad/shenzhen
[DIS]:https://github.com/nomad/shenzhen/tree/master/lib/shenzhen/plugins