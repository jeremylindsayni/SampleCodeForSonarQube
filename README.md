# SampleCodeForSonarQube

The outline steps for integrating the SonarQube static analysis tool into your Azure Pipeline are below:

## Create a DevOps project and add some code

1. Create a new DevOps project in your organisation. I choose to make this project public so I'm not using up my Pipeline build minutes.
2. When the project has been successfully created, browse to the "Azure Repos" section in the left hand nav.
3. From Azure Repos, choose to Import a Repository using the "Import" button. When challenged for a project URL, enter: https://github.com/jeremylindsayni/SampleCodeForSonarQube and click the "Import" button.

4. The project will need a few seconds to import the code from Git, but will eventually show a message saying "Import Successful".

## Create an Azure Pipeline

5. Now to create a simple pipeline - click on the "Azure Pipelines" section in the left hand nav.
6. There should be no pipelines - click on the "New Pipeline" button.
7. The next screen will ask "Where is your code?" - scroll to the bottom of the screen and choose "Use the classic editor".
8. The next screen will ask you to select a source - but the master branch of the repository you've imported into Azure Repos earlier will be selected by default. Click "Continue".
9. You'll be asked to choose a template. The code you've imported is an ASP.NET Core project - either scroll to the correct template, or type "ASP.NET Core" into the search box at the top right of the screen. The list of available templates will narrow down to just a couple. Select the "ASP.NET Core" template and click on the "Apply" button. DO NOT select the "ASP.NET Core (.NET Framework)" option.
10. After you click Apply, the default ASP.NET Core pipeline will be created, with stages like Restore, Build, Test, Publish and Publish Artifact.
11. Click on the "Save and queue" button in the top middle of the screen to save this pipeline, and start a new build. In the popup window, you leave the defaults as they are and just click on the "Save and run" button at the bottom right of the screen.
12. The pipeline will run successfully and complete after about a couple of minutes. There'll be one warning due to the fact there are no unit tests in this pipeline.

## Adding static analysis with SonarQube

13. Edit your pipeline by clicking on the button with three dots at the top right of the screen and select "Edit Pipeline".
14. Add a task to "Agent Job 1" by clicking on the "+" button, which is around the centre of the screen.
15. Type "SonarQube" into the search box of the nav that appears on the right hand side.
16. Select the first option, which should just be called "SonarQube", and click on the "Get it free" option to install the extension to your organisation.
17. A new window will open showing the extension on the Visual Studio marketplace - click on the "Get it free" button, and then click on "Install" in the next window. Once you see the "You're all set" message, you can close the window.
18. Back on the screen with the pipeline, click on "Refresh" in the right hand nav menu, and you'll see three new tasks available to you: Run Code Analysis, Prepare Analysis Configuration, and Publish Quality Gate Result.
19. Drag the "Prepare Analysis Configuration" task to be a new step between "Restore" and "Build".
20. You'll see a message saying "Some settings need attention". That's because we havent' set up a SonarQube server yet. Let's do that now.

## Create the SonarQube server

21. Browse to https://portal.azure.com and log in.
22. Click on the Cloud Shell icon in the top right menu (the icon looks like ">_" )
23. When the terminal has connected, run the command below - this will create a resource group:

az group create --name MySonarServer --location uksouth

24.Next create a SonarQube server using the official Docker image with the command below - remember to replace "YOUR_UNIQUE_SERVER_NAME" with something unique to you:

az container create -g MySonarServer --name sonarqubeaci --image sonarqube --ports 9000 --dns-name-label YOUR_UNIQUE_SERVER_NAME --cpu 2 --memory 3.5

Creating the image will take a couple of minutes, but after this runs you'll have a resource group running a SonarQube server.

## Configure the SonarQube server

25. Browse to your new SonarQube server to do some configuration - the URL will be something like:

http://YOUR_UNIQUE_SERVER_NAME.uksouth.azurecontainer.io:9000

26. When you see the SonarQube page, click on login - the default docker image credentials are username = Admin, password = Admin
27. After logging in, click on "Create new project" - in the next screen, choose a project name like "MySonarProject" and click "Setup".
28. The next screen will invite you to create a token - this will allow Azure Pipelines to communicate with your SonarQube server. Enter a name for your token e.g. TrainingToken and click on "Generate". Double click on the long token string generated, and copy this to your clipboard.

## Finish configuring Azure Pipelines.

29. Back in your Azure Pipelines, look at the "Prepare Analysis Configuration" task you just created. The section describing the SonarQube connection will be blank and highlighted in red. Click on the "+ New" button beside the highlighted section to add a connection.
30. In the window the pops up, paste in the token you copied in step 28 to the third box, enter in the server URL to the second box (remember it'll be something like http://YOUR_UNIQUE_SERVER_NAME.uksouth.azurecontainer.io:9000 ), and add in a unique name for you connection to the first box (more or less any text will do). Now click OK to save these settings.
31. Add two more tasks to your pipeline - click on the "+" button again to add tasks, and search for SonarQube again. 
32. Now drag the "Run Code Analysis" task to be after the "Test" task, and finally drag the "Publish Quality Gate Result" to be after the "Run Code Analysis" task.

## Run the pipeline and view your static analysis report

33. Now click on the "Save and Queue" button.

The pipeline will run again, but this time will send your code for static analysis on the SonarQube container we set up.

34. When the build completes, click on the Summary tab and scroll to the bottom. There will be a section called "SonarQube Analysis Report". Click on the "Detailed SonarQube report" link below the heading. This will open your analysis report in a new window, where you'll see all the bugs, possible vulnerabilities and code smells that SonarQube has detected.
