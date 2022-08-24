# Api Key

You can acquire your API key once you have logged in by going to the API page and clicking on the new button. All API calls must have a valid API key provided using a get query:
``https://api.myvar.cloud/build/4f90e126-edcc-4711-9fb0-385c9161c20b/status?key=55C5C10F610C0CC396523DB60A82C076``

# Example Script

```bash
#!/bin/bash

PROJECT_NAME="DemoProject"
API_KEY="01FE6FC27D7B84EF2B43D9A97E5A7A41"

#ensure deps are installed

function assertInstalled() {
    for var in "$@"; do
        if ! which $var &> /dev/null; then
            echo "$var Not Installed!"
            exit 1
        fi
    done
}

#ensure deps are installed
assertInstalled dotnet zip curl jq unzip awk

dotnet publish

p=$PWD
[ -e $p/$PROJECT_NAME.zip ] && rm $p/$PROJECT_NAME.zip

cd "./$PROJECT_NAME/bin/Debug/netcoreapp3.1/publish/"
zip -j -r $p/$PROJECT_NAME.zip . -i "*"
cd $p

TOKEN=$(curl -s https://api.myvar.cloud/build/create?key=$API_KEY |  jq --raw-output '.Body.BuildId')
echo "BUILD $TOKEN"

echo "Uploading"
UPLOAD_RAW=$(curl --request POST --data-binary "@$p/$PROJECT_NAME.zip" "https://api.myvar.cloud/build/$TOKEN/upload?key=$API_KEY")
UPLOAD_STATUS=$(echo $UPLOAD_RAW |  jq --raw-output '.Status')

if [ "$UPLOAD_STATUS" -eq "1" ];then
  echo "Upload Faild";
  echo "$($UPLOAD_RAW |  jq --raw-output '.Message')"
  exit 1
fi

echo "Upload was Successful";

echo "Starting build"
START_RAW=$(curl -s "https://api.myvar.cloud/build/$TOKEN/start?key=$API_KEY" )
START_STATUS=$(echo $START_RAW |  jq --raw-output '.Status')
if [ "$START_STATUS" -eq "1" ];then
  echo "Starting Faild";
  echo "$($START_RAW |  jq --raw-output '.Message')"
  exit 1
fi

echo "Start was Successful";
sleep 5
function pollStatus() {
 START_RAW=$(curl -s "https://api.myvar.cloud/build/$TOKEN/status?key=$API_KEY" )
 START_STATUS=$(echo $START_RAW |  jq --raw-output '.Body')
}
pollStatus
echo -n "Waiting for build to complete "
while [ "$START_STATUS" = "false" ]
do
    echo -n "."
    sleep 1
    pollStatus
done
echo ""
echo "Build Done"
echo "Downloading Build"
curl "https://api.myvar.cloud/build/$TOKEN/artifact?key=$API_KEY" --output build-artifact.zip

[ -e ./build/build.sh ] && rm -r ./build

unzip build-artifact.zip -d ./build
chmod 777 -R ./build
cd ./build
chmod +x build.sh
./build.sh

echo "you can now apply the Kubernetes spec file in the build directory"

cd ..

```

# Endpoints

## Create

``https://api.myvar.cloud/build/create``
Use this endpoint to create a new build, the body of the response json will contain a Guid for the specific build.

## Upload

``https://api.myvar.cloud/build/{guid}/upload``
Upload a zip of the publish folder in your projects bin folder after your ran the command ``dotnet publish``, the files in the zip must not be in a directory but rather at the root of the zip.

## Start

``https://api.myvar.cloud/build/{guid}/start``
This call will start the build.
## Status

``https://api.myvar.cloud/build/{guid}/status``
This call will return the status of your build.

## Artifact

``https://api.myvar.cloud/build/{guid}/artifact``
Once you have started the build and it has finished, you can download it from this url.

## Delete

``https://api.myvar.cloud/build/{guid}/delete``
Deletes both the uploaded binaries and artifacts related to this build.

