# Quick Start

An easy way to get familiar with 8base is to go through steps to download, install, and run a demo application. To make this process easy, we've created [a simple React app](https://github.com/8base/app-example) for you. It will help demonstrate some important concepts of 8base.

For context, the demo app is a nifty little real estate platform. It allows users to manage Brokers, Customers, Listings, and Properties, as well as offers a convenient "Share Listing" via email capability.

So... let's get to it!

## 1. Create an 8base Account

The first thing you'll need to do is create an 8base account. To get started, click [this link to signup](https://app.8base.com/auth/signup). If you use an email and password to create your account, we'll send you a confirmation email. Either way, once your account is confirmed 8base will redirect you to your default workspace.

![8base Signup](../.gitbook/assets/signup-screen%20%281%29.png)

Voila! You've officially signed up for 8base :\)

## 2. Visit your Workspace

In 8base, workspaces are treated like an individual project. Just like you probably have different Git repositories for different code bases. This allows you to easily stay organized with projects, as well as upgrade individual workspaces when your app or service begins to scale! Each workspace starts on a [30-day Free Trial](https://www.8base.com/pricing).

Lets just use the default workspace for the rest of this demo.

_Optional: If you want to create a new workspace, click the "YOUR NAME's Workspace" dropdown at the top of your screen and select "+ New Workspace". Name it whatever you like before pressing create. The new workspace should load in less then 10-seconds._

![Create a Workspace](../.gitbook/assets/workspace-menu%20%281%29.png)

## 3. Install the 8base CLI

Open up your Terminal of choice. To harness the full power of 8base, you'll need to have [Node.js](https://nodejs.org/) installed on your computer. Without it, you won't be able to install our handy [8base NPM package](https://www.npmjs.com/package/8base).

Assuming you're all set up and able to run `npm` commands, lets go ahead and install the 8base CLI.

```text
# Install 8base globally
npm install -g 8base
```

## 4. Clone the Demo App

8base gives you full freedom to use whatever front-end technologies you'd like! For this example though, we've built a simple demo app to expedite your learning. Try cloning it to you computer with the following commands.

```text
# Clone the app from GutHub
git clone https://github.com/8base/app-example.git

# Change into the app directory
cd app-example
```

## 5. Import and Deploy the Schema, Data and Backend Logic

As you may know by now, 8base gives developers a Serverless + GraphQL backend that's ready to rock'n roll from the get go. It's quick and convenient to update tables, fields, model relationships, and much more in the [8base console](https://app.8base.com/). Let's simply bootstrap our demo app with some definitions and data we've already prepared for you.

```text
# Change into the app-example/server directory
cd app-example/server

# Install required dependencies
npm install
```

Using the 8base CLI you'll be able to authenticate your development workspace. Allowing you to communicate with 8base for deploys, function invocations, logs, and more. Try running the following and allow your browser to launch a new window \(you may have to login\). If you have multiple workspaces set up already, the CLI will prompt you to select one. Also, make sure to note the API endpoint URL displayed after login - you will need it later.

```text
# Login with CLI.
8base login
```

![Logged in with 8base CLI](../.gitbook/assets/cli-login-success.png)

Now we're going to run the import using a `DEMO.json` file found in the `app-example/server` directory. Just so you know, this can take a minute.

```text
# Import data schema and sample data
8base import -f DEMO.json
```

Once the import is done... woohoo! You're ready to deploy using our handy `deploy` command. Once the deploy is finished, checkout the [Data Viewer](https://app.8base.com/data/) in your workspace. There should be brand new database tables filled with rows of data there waiting for you.

```text
# Deploy backend logic
8base deploy
```

![8base data viewer inside of workspace](../.gitbook/assets/demo-data-viewer.png)

## 6. Fire-up the App

Let's get this show on the road. To start up the app lets first change into the `app-example/client` directory. Just like before in `app-example/server` we'll first want to install all our dependencies here. Once that's handeled though, try running the following start command - the app may take a minute or two to build.

### For Mac Users

```text
# Change to client directory
cd ../client

# Install dependencies
npm install

# Swap placeholder with API URL and run start command
REACT_APP_8BASE_API_ENDPOINT=<YOUR_API_URL_GOES_HERE> npm start
```

![8base Demo Application Properties page](../.gitbook/assets/demo-app-properties%20%281%29.png)

### For Windows Users

If you're developing with 8base on a Windows machine, there are a few extra steps to take for everything to run smoothly.

1. In `app-example/client/package.json`, update the `react-scripts: '3.0.0'` dependency to be `react-scripts@2.1.8`
2. Install `npm install react-scripts@2.1.8` from your terminal
3. Install `npm install -g cross-env` from your terminal
4. Update `app-example/client/package.json` to only contain the following:
   * ```text
     {
      "parser": "babel-eslint",
     }
     ```
5. Run `cross-env REACT_APP_8BASE_API_ENDPOINT=<YOUR_API_URL_GOES_HERE> npm start` to start the app

## What Just Happened?

You can now login and logout of the demo app using your username and password. It's done! While you may have followed a few steps at this point, you probably haven't learned much about 8base. So, lets dive in and understand the fruits of our labor.

### 8base.yml

If the `app-example` is not already open in your favorite text editor, open it now and navigate to the `app-example/server` directory. Inside you'll find three really important files. They are the files marked with an astrix \(\*\) in the following print-out:

```text
server/
├── node_modules/
├── src/
│   ├── mailer/
│   └── mutations/
│       └── listingShare/
│           ├── handler.js*
│           └── schema.graphql*
├── 8base.yml*
├── DEMO.json
├── package-lock.json
└── package.json
```

By running the `8base deploy` command you deployed a custom function to the serverless cloud. There are many custom function types you can deploy on 8base, all of which must be configured in the `8base.yml` file located at the root of your project. By looking at that file now, we can see a functions declaration containing a `listingShare` object that has three important keys - _handler_, _type_, and _schema_.

```yaml
functions:
  listingShare:
    handler:
      code: src/mutations/listingShare/handler.js
    type: resolver
    schema: src/mutations/listingShare/schema.graphql
```

In this quickstart, we'll skip the different function types. Just know that your `8base.yml` file is the master config file of your serverless application. In it we must declare any path to our functions and their supporting files. In the case of this `resolver` function, we've effectively extended the GraphQL API with a new Mutation that handles the "Share Listing" feature we mentioned earlier. Take a moment to check out the `handler.js` and `schema.graphql` files in `src/mutations/listingShare`. All the magic happens there :\)

### 8base Data Viewer

Your workspace was provisioned with four custom database tables upon importing the `DEMO.json` file - each with a dozen or more rows of data. The most popular way to accomplish this from scratch is by using the [Data Viewer](https://app.8base.com/data/). Navigate there now and you'll see how easy it is to create, update, and delete your table schemas, as well as view the data they contain by switching over to the **Data** tab.

Just for fun, lets add an _Address_ field to your **Properties** table.

1. Click on **Properties**
2. Type "Address" into the _Add New Field_ input
3. Select "Smart" from the "Select Data Type" dropdown
4. Once the field form opens up, select "Address" as the format
5. Create the field

![Creating fields in 8base](../.gitbook/assets/add-field%20%281%29.png)

As simple as that, your _Properties_ table can now save addresses.

What's very important to know is that 8base creates all basic, and some advance, GraphQL _queries_, _mutations_, and _subscriptions_ auto-magically for every table you define. What does this mean? You'll never again have to write another Create, Read, Update, or Delete \(CRUD\) resource / endpoint!

Don't believe me? Check out the next section.

### API Explorer

As promised, all of your CRUD actions are right here and ready to go. Let's prove it - as well as see all the GraphQL access that your applications now have access to.

Navigate to the [8base API Explorer](https://app.8base.com/api-explorer). This is essentially your API playground, where you can quickly develop and execute powerful GraphQL commands. Copy over the following command once the API explorer is loaded.

> Always be careful! The API explorer makes use of your live production data.

```javascript
query {
  propertiesList(first: 10) {
    items {
      title
      description
      pictures {
        count
      }
    }
  }
}
```

Run the query by pressing the large _play_ button. Your requested _Properties_ list will pop up in a blink! Only sharing with you the requested data of _title_, _description_, and _pictures\[count\]_.

![Running queries in API Explorer](../.gitbook/assets/data-query%20%281%29.png)

There is so much more you can do and learn in the API Explorer about GraphQL and your data, so we suggest taking some time to experiment.

## Conclusion

We hope this guide helps you better understand how 8base works. Feel free to modify the data schema in your workspace, add new tables, deploy custom logic, and develop amazing applications using 8base.
