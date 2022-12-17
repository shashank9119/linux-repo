## AWS Beanstalk sample application steps


1) Java installation 
    
    a) Install java 
        
        https://openjdk.org/install/
    
    b) Verify java installation 
        
        java --version

2) Create the source code

3) Download and install Maven.
    
    a) Download and install Maven
    
        https://maven.apache.org/download.cgi

    
    b) Extract tar file 
    ```
        tar -xvf apache-maven-3.6.3-bin.tar.gz
        export M2_HOME="/Users/pankaj/Downloads/apache-maven-3.6.3"
        PATH="${M2_HOME}/bin:${PATH}"
        export PATH
        mvn -version
    ```


4) Switch to an empty directory on your local computer or instance, and then run this Maven command.
    ```
    mvn archetype:generate "-DgroupId=com.mycompany.app" "-DartifactId=ROOT" "-DarchetypeArtifactId=maven-archetype-webapp" "-DinteractiveMode=false"`
    ```
5) Create a subdirectory named `.ebextensions` in the ROOT directory. 
n the .ebextensions subdirectory, create a file named `fix-path.config` with this content

    ```
    container_commands:
      fix_path:
        command: "unzip ROOT.war 2>&1 > /var/log/my_last_deploy.log"
    ```


Scenario B: Use CodePipeline to run CodeBuild and deploy to Elastic Beanstalk

 In this scenario, you complete the steps to prepare and upload the source code. You create a build project with CodeBuild and an Elastic Beanstalk application and environment with the AWS Elastic Beanstalk console. You then use the AWS CodePipeline console to create a pipeline. After you create the pipeline, CodePipeline builds the source code and deploys the build output to the environment.



Step b1: Add a buildspec file to the source code

In this step, you create and add a buildspec file to the code you created in Create the source code. You then upload the source code to an S3 input bucket or a CodeCommit, GitHub, or Bitbucket repository.

a) Create a file named buildspec.yml with the following contents. Store the file in the ROOT directory.

```
version: 0.2             
phases:
  install:
    runtime-versions:
      java: corretto11
  post_build:
    commands:
      - mvn package
      - mv target/ROOT.war ROOT.war
artifacts:
  files:
    - ROOT.war
    - .ebextensions/**/*
```


 b) Your file structure should now look like this.

            └── ROOT
                ├── .ebextensions
                │   └── fix-path.config
                ├── src
                │   └── main
                │       ├── resources
                │       └── webapp
                │           ├── WEB-INF
                │           │   └── web.xml
                │           └── index.jsp
                ├── buildpsec.yml
                └── pom.xml

    c) Upload the contents of the ROOT directory to GitHub (not root dir itself).


Step b2: Create a build project

 In this step, you create an AWS CodeBuild build project to use with your pipeline.

 1) Open the AWS CodeBuild console at https://console.aws.amazon.com/codesuite/codebuild/home.

 2) Create a build project. For more information, see Create a build project (console) and Run a build (console). Leave all settings at their default values, except for these settings.

    For Environment:

        For Environment image, choose Managed image.
        For Operating system, choose Amazon Linux 2.
        For Runtime(s), choose Standard.
        For Image, choose aws/codebuild/amazonlinux2-x86_64-standard:3.0.

    For Artifacts:

        For Type, choose Amazon S3.
        For Bucket name, enter the name of an S3 bucket.
        For Name, enter a build output file name that's easy for you to remember. Include the .zip extension.
        For Artifacts packaging, choose Zip.


Step b3: Create an Elastic Beanstalk application and environment
    
    In this step, you create an Elastic Beanstalk application and environment to use with CodePipeline.

    1) Open the Elastic Beanstalk console at https://console.aws.amazon.com/elasticbeanstalk/.

    2) Use the AWS Elastic Beanstalk console to create an application. For more information, see Managing and configuring AWS Elastic Beanstalk applications in the AWS Elastic Beanstalk Developer Guide.

    3) Use the AWS Elastic Beanstalk console to create an environment. For more information, see The create new environment wizard in the AWS Elastic Beanstalk Developer Guide. Except for Platform, leave all settings at their default values. For Platform, choose Tomcat.

Step b4: Create the pipeline and deploy
    
 In this step, you use the AWS CodePipeline console to create a pipeline. After you create and run the pipeline, CodePipeline uses CodeBuild to build the source code. CodePipeline then uses Elastic Beanstalk to deploy the build output to the environment.



    1) Create or identify a service role that CodePipeline, CodeBuild, and Elastic Beanstalk can use to access resources on your behalf. For more information, see Prerequisites.

    2) Open the CodePipeline console at https://console.aws.amazon.com/codesuite/codepipeline/home.

        Use the AWS Region selector to choose an AWS Region where CodeBuild is supported. If you're storing the source code in an S3 input bucket, the output bucket must be in the same AWS region as the input bucket.

    3) Create a pipeline. For information, see Create a pipeline that uses CodeBuild (CodePipeline console). Leave all settings at their default values, except for these settings.

        On Add build stage, for Build provider, choose AWS CodeBuild. For Project name, choose the build project you just created.

        On Add deploy stage, for Deploy provider, choose AWS Elastic Beanstalk.

            For Application name, choose the Elastic Beanstalk application you just created.

            For Environment name, choose the environment you just created.
