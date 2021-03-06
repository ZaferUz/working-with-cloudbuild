# working-with-cloudbuild


## Task 1: Confirm that needed APIs are enabled

- Make a note of the name of your Google Cloud project. This value is shown in the top bar of the Google Cloud Console. It will be of the form qwiklabs-gcp- followed by hexadecimal numbers.

- In the Google Cloud Console, on the Navigation menu(Navigation menu), click APIs & Services.

- Click Enable APIs and Services.

- In the Search for APIs & Services box, enter Cloud Build.

- In the resulting card for the Cloud Build API, if you do not see a message confirming that the Cloud Build API is enabled, click the ENABLE button.

- Use the Back button to return to the previous screen with a search box. In the search box, enter Container Registry.

- In the resulting card for the Container Registry API, if you do not see a message confirming that the Container Registry API is enabled, click the ENABLE button.

## Task 2. Building Containers with DockerFile and Cloud Build

- You can write build configuration files to provide instructions to Cloud Build as to which tasks to perform when building a container. These build files can fetch dependencies, run unit tests, analyses and more. In this task, you'll create a DockerFile and use it as a build configuration script with Cloud Build. You will also create a simple shell script (quickstart.sh) which will represent an application inside the container.

- On the Google Cloud Console title bar, click Activate Cloud Shell.

- When prompted, click Continue.

- Cloud Shell opens at the bottom of the Google Cloud Console window.

- Create an empty quickstart.sh file using the nano text editor.

```
nano quickstart.sh

```
-- Add the following lines in to the quickstart.sh file:

```
#!/bin/sh
echo "Hello, world! The time is $(date)."


```

- Save the file and close nano by pressing the CTRL+X key, then press Y and Enter.

- Create an empty Dockerfile file using the nano text editor.

```
 nano Dockerfile

```
- Add the following Dockerfile command:

```
FROM alpine
# This instructs the build to use the Alpine Linux base image.

# Add the following Dockerfile command to the end of the Dockerfile:

COPY quickstart.sh /
# This adds the quickstart.sh script to the / directory in the image.

# Add the following Dockerfile command to the end of the Dockerfile:

CMD ["/quickstart.sh"]
# This configures the image to execute the /quickstart.sh script when the associated container is created and run.

```
- The Dockerfile should now look like:

```
FROM alpine
COPY quickstart.sh /
CMD ["/quickstart.sh"]

```
- Save the file and close nano by pressing the CTRL+X key, then press Y and Enter.

- In Cloud Shell, run the following command to make the quickstart.sh script executable.
```
chmod +x quickstart.sh
```
- In Cloud Shell, run the following command to build the Docker container image in Cloud Build.
```
gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/quickstart-image .
```
--Important: Don't miss the dot (".") at the end of the command. The dot specifies that the source code is in the current working directory at build time.

- When the build completes, your Docker image is built and pushed to Container Registry.

- In the Google Cloud Console, on the Navigation menu (Navigation menu), click Container Registry > Images.

- The quickstart-image Docker image appears in the list

## Task 3. Building Containers with a build configuration file and Cloud Build
- Cloud Build also supports custom build configuration files. In this task you will incorporate an existing Docker container using a custom YAML-formatted build file with Cloud Build.

- In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.
```
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```
- Create a soft link as a shortcut to the working directory.
```
ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s
```
- Change to the directory that contains the sample files for this lab.
```
cd ~/ak8s/Cloud_Build/a
```
- A sample custom cloud build configuration file called cloudbuild.yaml has been provided for you in this directory as well as copies of the Dockerfile and the quickstart.sh script you created in the first task.

- In Cloud Shell, execute the following command to view the contents of cloudbuild.yaml.
```
cat cloudbuild.yaml
```
- You will see the following:
```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
```
- This file instructs Cloud Build to use Docker to build an image using the Dockerfile specification in the current local directory, tag it with gcr.io/$PROJECT_ID/quickstart-image ($PROJECT_ID is a substitution variable automatically populated by Cloud Build with the project ID of the associated project) and then push that image to Container Registry.

- In Cloud Shell, execute the following command to start a Cloud Build using cloudbuild.yaml as the build configuration file:

```
gcloud builds submit --config cloudbuild.yaml .
```
- The build output to Cloud Shell should be the same as before. When the build completes, a new version of the same image is pushed to Container Registry.

- In the Google Cloud Console, on the Navigation menu (Navigation menu), click Container Registry > Images and then click quickstart-image.


- Two versions of quickstart-image are now in the list.

- Build two Container images in Cloud Build.

- In the Google Cloud Console, on the Navigation menu (Navigation menu), click Cloud Build > History.


- Two builds appear in the list.

- Click the build ID for the build at the top of the list.

- The details of the build, including the build log, are displayed.

## Task 4. Building and Testing Containers with a build configuration file and Cloud Build
- The true power of custom build configuration files is their ability to perform other actions, in parallel or in sequence, in addition to simply building containers: running tests on your newly built containers, pushing them to various destinations, and even deploying them to Kubernetes Engine. In this lab, we will see a simple example: a build configuration file that tests the container it built and reports the result to its calling environment.

- In Cloud Shell, change to the directory that contains the sample files for this lab.
```
cd ~/ak8s/Cloud_Build/b
```
- As before, the quickstart.sh script and the a sample custom cloud build configuration file called cloudbuild.yaml has been provided for you in this directory. These have been slightly modified to demonstrate Cloud Build's ability to test the containers it has build. There is also a Dockerfile present, which is identical to the one used for the previous task.

- In Cloud Shell, execute the following command to view the contents of cloudbuild.yaml.
```
cat cloudbuild.yaml
```

- You will see the following:
```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
- name: 'gcr.io/$PROJECT_ID/quickstart-image'
  args: ['fail']
images:
- 'gcr.io/$PROJECT_ID/quickstart-image
```
- In addition to its previous actions, this build configuration file runs the quickstart-image it has created. In this task, the quickstart.sh script has been modified so that it simulates a test failure when an argument ['fail'] is passed to it.

- In Cloud Shell, execute the following command to start a Cloud Build using cloudbuild.yaml as the build configuration file:
```
gcloud builds submit --config cloudbuild.yaml .
```
- You will see output from the command that ends with text like this:

- Output (do not copy)
```
Finished Step #1
ERROR
ERROR: build step 1 "gcr.io/ivil-charmer-227922klabs-gcp-49ab2930eea05/quickstart-image" failed: exit status 127
----------------------------------------------------------------------------------------------------------------------------------------------------------------
ERROR: (gcloud.builds.submit) build f3e94c28-fba4-4012-a419-48e90fca7491 completed with status "FAILURE"
```
- Confirm that your command shell knows that the build failed:
```
echo $?
```
The command will reply with a non-zero value. If you had embedded this build in a script, your script would be able to act up on the build's failure.


