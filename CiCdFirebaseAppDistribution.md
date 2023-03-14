# CI/CD ****Firebase App Distribution****

**CI/CD** is simply continuous integration and continuous delivery or continuous deployment. Automating the most routine activities when developing the next billion-user application or software is essential and gives you time to focus on other things.

create a folder at the root level of your project. “***.github/workflows”*** 
ensure you name it as required.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28c1575e-eddb-4c83-a5c7-cc72af84279a/Untitled.png)

A workflow is a configurable automated process that will run one or more jobs, defined by a YAML file. So, create a file <filename>.yml inside the workflows folder and name it something in relation to your action. *eg: flutter.yml*

Github action consists of parts:

1. **Events:** for an action to be triggered, we are required to state the condition such as when there is a new pull request, issue, pull, etc. Link to [All available events](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#available-events)
2. **Jobs:** When an event occurs, the jobs simply state what is to be done. You could have multiple jobs triggered when an event occurs, jobs are built to run parallel, but you could configure a job to wait for a job to be done before executing.
3. **Runners:** this is simply the machine on which we will be running our actions. Github action has already set up VMs at could be used by simply specifying the machine you want to use or testing across different machines (macOS, ubuntu and windows) and also providing you with an option of setting up your own machine.
4. **Steps**: Just as the word implies, the steps to carrying out the job
5. **Action**: Finally the action or commands (eg: *run: flutter build apk —* to build and apk file)

Let’s write YAML and see how everything comes together:

```dart
name: CI/CD

on: [pull_request, push]

jobs:   
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - uses: actions/setup-java@v1
        with:
          java-version: '12.x'
      - uses: subosito/flutter-action@v1
        with:
          channel: 'stable'

      - name: Build Gradle
        run: flutter build apk --release
      - run: ls build/app/outputs/flutter-apk

      # - run: ls build/app/outputs/bundle/release
      - name: Upload artifact to Firebase App Distribution
        uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId: ${{secrets.APP_ID}}
          serviceCredentialsFileContent: ${{secrets.CREDENTIAL_FILE_CONTENT}}
          groups: testers
          file: build/app/outputs/flutter-apk/app-release.apk
```

Breaking the script down:

*name:* Something descriptive to explain what the file does.

We said when a *push or PR request* is made to our branch *main* (that the event)*,* we want the below *jobs* executed.

Next, we put down the *steps:*

We set our runner with *runs on:* then specify the machine we need

We specify the version with the *uses:* in our example, we specified the java and flutter versions we want to use.

Finally, we pass in the actions with *run:* , You have the option of giving your actions a descriptive name. with the *— name:*

Now, the GitHub action is automatically triggered whenever a push or pull request is made to the *main* branch and it is as easy as that setting up action.

Head over to [firebase console](https://firebase.google.com/) and create a project if you've not done that yet. If this is your first project with firebase, you will need to install firebase on your local machine.

Head over [here](https://firebase.google.com/docs/cli#setup_update_cli) to get a more detailed guide to installing firebase CLI, but let's go over it here: Open up your terminal and run

```
npm install -g firebase-tools
```

After that has ran a successfully installation, login to your firebase account.

```
firebase login
```

Now, over to your flutter project, open up the terminal and install flutterfire_cli

```
dart pub global activate flutterfire_cli
```

ensure you add the installation to your machine env. Configure your project by running

```
flutterfire configure
```

If you get an error like *command not found: flutterfire* it means you’ve not added your installation to env and your machine doesn’t know you have flutterfire CLI installed. If you’ve installed everything right. The above command should display the list of projects you have on your console and an option to create a new project. If you have an existing project, simply select it. If not create a new project and follow the prompts and you should have a project up and running in no time on firebase.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f3a108a4-4df3-49b2-b567-d5b625b3c3f7/Untitled.png)

So, *flutterfire configure* syncs your project to firebase and adds the required dependencies and file (firebase_options.dart) to your project.Next, you are required to add the firebase_core package to your pubspec.yaml to clear the error in the firebase_options file.

```
flutter pub add firebase_core
```

and add the below code in the main.dart file in the *main* method

```

void main() async {
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}
```

Github action is set to build and publish our app apk to firebase, but we need to set up our APP_ID and CREDENTIAL_FILE_CONTENT secret for line 32 and 33.

Head over to the repo > settings > secret and variable > Actions > Click New Repository Secret

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/55fa0e75-751b-45dd-89ed-ed370dc8c92c/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/491298dc-8589-4736-a026-b989c07dd8cc/Untitled.png)

Add your app id from firebase > to get the APP_ID, we could get this in two methods:

**Method 1**: Go to your firebase_options.dart file, open it and scroll till you see a code like below. Copy the app id and paste it.

```
static const FirebaseOptions android = FirebaseOptions(
    apiKey: 'AIzaSy******************',
    appId: '1:2********:android:8**********', //your app id
    messagingSenderId: '23454676532',
    projectId: 'cicd-guide-01',
    storageBucket: '',
  );
```

**Method 2**: Open the firebase project > settings > project settings > your app id should be at the bottom just scroll down.

![https://miro.medium.com/max/1188/1*nbKdKWbNHlzexmNI_eq_TQ.png](https://miro.medium.com/max/1188/1*nbKdKWbNHlzexmNI_eq_TQ.png)

![https://miro.medium.com/max/3214/1*MM_-WL9xONEoACqD7HLR2Q.png](https://miro.medium.com/max/3214/1*MM_-WL9xONEoACqD7HLR2Q.png)

Previously we could use firebase ***token*** to authorize out application to interact with firebase but that is now deprecated. Hence, we will be using ***serviceCredentialFileContents**.* To get this file you need to go to Google cloud [https://console.cloud.google.com](https://console.cloud.google.com/welcome?project=credit-sparrow)

By default, all your firebase projects should also be accessible in your cloud account.

Click on All > and select the project you’re currently integrating with.

![https://miro.medium.com/max/700/1*A1VkJ-fnF9blEMbvxgk3lw.png](https://miro.medium.com/max/700/1*A1VkJ-fnF9blEMbvxgk3lw.png)

![https://miro.medium.com/max/3200/1*IZF1KxdqV_wSsu57ptcCFg.png](https://miro.medium.com/max/3200/1*IZF1KxdqV_wSsu57ptcCFg.png)

![https://miro.medium.com/max/1072/1*DvQAFqlibGNG4T2PRPkz9w.png](https://miro.medium.com/max/1072/1*DvQAFqlibGNG4T2PRPkz9w.png)

Click on Service Account > Create Service Account > give the service account a name. and create.

![https://miro.medium.com/max/700/1*egDQJFkU81TbbUfvKSQ6pw.png](https://miro.medium.com/max/700/1*egDQJFkU81TbbUfvKSQ6pw.png)

grant service permission to firebase app distribution and click on DONE

![https://miro.medium.com/max/700/1*vtGFIkHZh4ZVNprOcK9z5w.png](https://miro.medium.com/max/700/1*vtGFIkHZh4ZVNprOcK9z5w.png)

A list of service should be displayed now. The one named with your project select the : by the right end. Click on Manage Keys and create a new key

![https://miro.medium.com/max/700/1*_XFvxJvNgOkOi8I8DSWEAw.png](https://miro.medium.com/max/700/1*_XFvxJvNgOkOi8I8DSWEAw.png)

Ensure the key type is JSON and create

![https://miro.medium.com/max/700/1*rzThnjxrI69nXw_rB-cIdw.png](https://miro.medium.com/max/700/1*rzThnjxrI69nXw_rB-cIdw.png)

A JSON file should automatically be downloaded. Open it up, and copy everything in it. Head back to GitHub and paste it in secret and you’re done.

![https://miro.medium.com/max/700/1*uo9Hs8_lLRt5O7evMkCNvw.png](https://miro.medium.com/max/700/1*uo9Hs8_lLRt5O7evMkCNvw.png)

You do not have to include the download file in your project structure. Just keep it somewhere safe 😉

On Firebase Console left pan. Click on Release and monitor > App Distribution and create.

App Distribution interface Firebase

![https://miro.medium.com/max/3178/1*-MODra2eNOsVSTBBrdHQTA.png](https://miro.medium.com/max/3178/1*-MODra2eNOsVSTBBrdHQTA.png)

![https://miro.medium.com/max/3200/1*Eif8QDwfS3VaYWKc6e_duQ.png](https://miro.medium.com/max/3200/1*Eif8QDwfS3VaYWKc6e_duQ.png)

You have can create Testing groups and add different testers to the group. I will let you explore that on your own. Create a group and add it to groups in the Github action file Line 34 in the code above.

You should be all set now. Push your code fixes to github and let github action build and push to firebase.

![https://miro.medium.com/max/700/1*kwBHRhUIRpGH5eFgbDwMRg.jpeg](https://miro.medium.com/max/700/1*kwBHRhUIRpGH5eFgbDwMRg.jpeg)

**Finally update app version from pubspec.yaml**