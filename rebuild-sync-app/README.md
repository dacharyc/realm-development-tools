# Rebuild Realm Sync Applications

When you develop Realm Sync applications, 
[breaking schema changes](https://www.mongodb.com/docs/realm/sync/data-model/update-schema/#breaking-vs.-non-breaking-change-quick-reference)
require you to terminate and re-enable Sync. They are also incompatible with existing
data. During development, you might want to delete the DB from MongoDB Atlas,
delete the Realm app, and then create a new Realm App with the same configuration. 
This script automates that process. Its output includes the new Realm App ID you need to replace 
in your project.

IMPORTANT: This script deletes all of the data in your MongoDB Atlas DB, and all of the
users and logs in your Realm App backend, as well as the Realm App backend itself. 
Only use this script in development when you do not need to save important data or user info.

## Prerequisites

### Clone the repo or copy the script and dropDB.js files

Clone the repo to your machine to get your very own
copy of the script, and potentially pull in any future updates 
(or PR the script with improvements!) Alternately, you can copy-and-paste the contents
of the script into a file on your machine. If you do copy-and-paste, make sure to also
create a copy of `dropDB.js` in the same directory.

### Install mongosh and Realm CLI

To use this script, you must have installed:

- [mongosh](https://www.mongodb.com/docs/mongodb-shell/install/)
- [Realm CLI](https://www.mongodb.com/docs/realm/cli/#installation)

After installing Realm CLI, you must use `realm-cli login` to authenticate with it.
For more info, see: [realm-cli login](https://www.mongodb.com/docs/realm/cli/realm-cli-login/).

### Replace the Atlas Connection String with your connection string

You must connect to Atlas to drop the DB. To connect to Atlas, `mongosh` needs
an authenticated user.

You can find your connection string in Atlas. 

1. In your Atlas project, click the **Atlas** tab next to your project name. 
2. In the **Database Deployments** pane, press the **Connect** button next to your cluster name.
3. Select the **Connect with the MongoDB Shell** option. This connection string looks something like:

   ```
   mongosh "mongodb+srv://cluster0.s8qcn.mongodb.net/someDatabase" --apiVersion 1 --username someUsername
   ```

The connection string that Atlas gives you does not include the password. You'll get a prompt to enter
your database password when you run the script. This password does not persist outside of this script
run; it isn't stored on your file system for security reasons. You must enter it every time you
run the script.

Alternately, you could remove lines 4 and 5 from the script and rename the `$PASSWORD` in line 9 
to match some environment variable you use to save your password to your machine. This is less secure
because it involves saving your password in plain text to a file on your machine. Not recommended.

For more info about MongoDB connection strings, see: [Connect to a Deployment -> MongoDB on a Remote Host](https://www.mongodb.com/docs/mongodb-shell/connect/#mongodb-instance-on-a-remote-host).

IMPORTANT: The DB User in your connection string must have the `atlasAdmin@admin` role.
Otherwise, the user cannot drop the database. For more about project access roles, see:
[Project Roles](https://www.mongodb.com/docs/atlas/reference/user-roles/#project-roles).

For info on how to change a user's role, see: [Manage Access to a Project](https://www.mongodb.com/docs/atlas/access/manage-project-access/).

### Download and set Realm App configuration file directory as an environment variable

To recreate your Realm App, you must have a local copy of your Realm App configuration files. Use the 
`realm-cli pull` command to export a copy of your configuration files to a local directory.
For more details, see: [realm-cli pull](https://www.mongodb.com/docs/realm/cli/realm-cli-pull/).

Store the local directory as an environment variable named `REALM_APP` in your .bashrc or equivalent file.

In .bashrc, this looks something like:

```
export REALM_APP=`~/workspace/realm-app-config-myapp`
```

Although this should point to your app name and the path where you saved the config directory.

### Add the app ID you want to delete as an environment variable

This script needs the Realm App ID for the existing Realm App backend that you want to delete.
Save the id as an environment variable named `APP_ID` in your .bashrc or equivalent file.

In .bashrc, this looks something like:

```
export APP_ID="prfocus-tgslu"
```

Where the name in quotes is your Realm App ID.

For help finding your app ID, see: [Find a Realm Application ID](https://www.mongodb.com/docs/realm/get-started/find-your-project-or-app-id/#find-a-realm-application-id)

## Run the script

Note: If you made changes to your `.bashrc` above, don't forget to source .bashrc 
before you run the script. This looks something like:

```
source ~/.bashrc
```

Where the directory path points to your `.bashrc` file on your machine.

To run the script, go to the directory that contains the script and type:

`./rebuild-sync-app`

The output of the script includes the line:

```
The new app ID is: [YOUR NEW APP ID]
```

This is the new app ID for your client code.

This script comes with a `dropDB.js` file. This file only contains one line;
the command `db.dropDatabase()` that `mongosh` needs to execute to drop the DB. 
In order to avoid running `mongosh` interactively, we include this command in a .js
file that we load in line 9 of the script.

If you move the `rebuild-sync-app` file to another directory, you'll need to also 
move `dropDB.js` or edit the script to use a relative path for it.

### Interactivity

While the script is running, you'll need to respond to a prompt:

```
Enter your Atlas db password:
```

If you do not respond to this prompt, the script cannot complete.

The request for your password appends your Atlas DB user password to the connection
string in line 9. This password does not exist outside of the script and is not
persisted to your file system.

## What the script does

Here's a breakdown line-by-line of what the script does, to help with potential troubleshooting:

### Ln 3
Search the directory you specified when you set the directory of your 
existing Realm App configuration files as an environment variable. The script looks 
for the name of your existing Realm App in the `realm_config.json` in this directory.
It uses this name later when it creates a new Realm App.

### Ln 4
Display a prompt asking you to enter your Atlas db password. This password exists
only for the scope of this script run and does not persist to your file system.

### Ln 5
Read the user input to a variable called `PASSWORD`. The `-s` hides the input as 
you paste in your password.

### Ln 6
Use Realm CLI to delete the existing Realm App. This is the app whose ID you saved
as an environment variable called `APP_ID` earlier.

### Ln 8
This is here in case you didn't read the instructions and aren't sure what to do
with ln 9. Delete it.

### Ln 9
This line does a lot of heavy lifting. It's the line that drops the database from
your Atlas cluster. Breaking it down, `mongosh` invokes mongosh - what is normally
an interactive shell that lets you manage your MongoDB deployments. You can use this
to test queries and perform operations directly with your DB. Here, we're using it
to drop the database that has your existing Realm Sync data.

THIS DELETES YOUR DATA.

Presumably you're ok with this since your app is in development and you're iterating.
You'd have to jump through a lot of hoops to use this data after making breaking schema
changes, so it's probably just easier to delete it while your app is in development.

The next part of ln 9 - everything that's in square brackets - is your Atlas connection 
string. It should include the name of the database you want to drop, the API version, 
and your Atlas DB username. Replace this entire section, including the square brackets,
with your project's real Atlas connection string.

After the connection string, the script passes the `--password` flag with the password
data that you endered in ln 5 as the `$PASSWORD` variable. If you were running mongosh
interactively, you wouldn't need this password as you would be prompted to enter it
after providing the connection string. Since we want to run this as a script here
and not an interactive shell, we can pass the password as an option.

Finally, there's the `--file` flag with `dropDB.js`. This is a .js file that contains
one line - the command `db.dropDatabase()` - which you would normally execute inside
the interactive shell that is `mongosh`. Since we want to script this instead of 
working with the interactive shell, we save it to a .js file and pass it to `mongosh`
to execute for us.

### Ln 10
This line is here and commented out to demonstrate what ln 9 should look like.
It's just a helper and you can feel free to delete it.

### Ln 12
At its core, this line uses Realm CLI to create a new Realm App, with the
same name as your old Realm App, which we got up above in line 3. The `--local temp`
option is here because if you don't give this command a place to put the config files 
for your new Realm App, it makes its own directory. Then, when you make it again, 
this script will hang forever at this step because there is already a directory
at that location, and the CLI is waiting for you to respond to a prompt with a warning 
and an option to specify a new location. We'll delete this `temp` directory 
in the next line so it won't interfere with future runs.

The `REALM_CLI_CREATE_APP_OUTPUT` at the beginning of this line captures the 
output of the `realm-cli apps create` command so it can grab your new app ID.

### Ln 13
Remove the temporary directory we created in line 12 to hold the new Realm App config 
files. We don't need them, because we're overwriting the config with your existing
Realm App config files.

### Ln 14
Pipe the output from the `realm-cli create app` command to grep, and search
for the `client_app_id` of your new Realm App. Store this as `NEW_APP_ID`.

### Ln 15
Use Realm CLI to update the new Realm App we created with the existing Realm App
config files you stored as an environment variable earlier. This pushes 
the config containing your Sync details, permissions, authentication methods,
and other relevant Realm App config details. The `--yes` flag here answers yes
automatically to a prompt that would ask you to confirm the update if you were
running this interactively.

### Ln 16
Show the new app ID in the terminal, so you can grab it and use it in your project.
Look for the line `The new app ID is:` in the terminal output. Replace the old
app ID in your client application code, and you should be able to connect to your 
new Realm App backend!

## Output

Assuming everything goes successfully, this is what the output looks like:

```
Enter your Atlas db password:
Successfully deleted 1/1 app(s)
  ID                        Name     Deleted  Details
  ------------------------  -------  -------  -------
  625c9...................  PRFocus  true
Current Mongosh Log ID:	625c9...................
Connecting to:		mongodb+srv://cluster0.s8qcn.mongodb.net/SampleTestDB?appName=mongosh+1.3.1
Using MongoDB:		5.0.7 (API Version 1)
Using Mongosh:		1.3.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

Loading file: dropDB.js
Determining changes
Creating draft
Pushing changes
Deploying draft
Deployment complete
Successfully pushed app up: sample-tester-tgslu
The new app ID is: sample-tester-tgslu
```

## Troubleshooting

A future version of this script will include some error handling, but here are some 
possible issues if you need to debug problems in the meantime:

### Have you installed mongosh and realm-cli?

You need both of these tools when running this script. At the time of this
script's development, it worked with:

realm-cli version 2.4.2
mongosh version 1.3.1

### Have you authenticated with realm-cli?

You must generate an API key and login with realm-cli before you can run this script.
For more info, see: [realm-cli login](https://www.mongodb.com/docs/realm/cli/realm-cli-login/).

### Does your Realm-CLI user have permissions to delete and create apps?

If your Realm-CLI user API key is for a read-only user, you don't have sufficient permissions
to create and delete apps. 

For information on how to generate a new API key or update an existing one, see: 
[Realm CLI Authenticate with API Token](https://www.mongodb.com/docs/realm/reference/cli-auth-with-api-token/).

### Have you replaced the contents of the square brackets in ln 9 with your Atlas connection string?

Ln 9 needs your specific Atlas connection string to drop the database. Make sure you replace
the contents of the [], including the [] themselves, with your connection string.

### Did you replace the DB in the Atlas connection string with the DB you want to drop?

When you get the connection string from Atlas, it includes a placeholder for the datbase:

```
"mongodb+srv://cluster0.s8qcn.mongodb.net/myFirstDatabase"
```

Make sure you replace the **myFirstDatabase** at the end of the string with the actual 
name of the DB you want to delete from Atlas.

### Does your user have permissions to drop the database?

The DB user whose username you use in the connection string in ln 9 must have sufficient permissions
to drop the database. 

The DB User in your connection string must have at minimum the `atlasAdmin@admin` role.
For more about project access roles, see: [Project Roles](https://www.mongodb.com/docs/atlas/reference/user-roles/#project-roles).

For info on how to change a user's role, see: [Manage Access to a Project](https://www.mongodb.com/docs/atlas/access/manage-project-access/).

### Did Realm CLI fail to delete the app?

If Realm CLI was not able to delete the app, you might not have provided the `APP_ID` variable that contains
the Realm App ID you want to delete. Alternately, you may not have sourced the .bashrc or otherwise given the 
script access to this variable.

### Did Realm CLI fail to create the app?

Realm-CLI might have trouble creating the app if the directory already exists where
the app creation tries to place your new app's files. It may be trying to prompt you inside
the script. Try running the realm-cli app create command in ln 12 manually (just the part inside the brackets)
and see if it works. You'll need to replace `$APP_NAME` with the string name you want to use for your new app.
If successful, the output should look something like this:

```
Successfully created app
{
  "client_app_id": "prfocus-tgslu",
  "filepath": "/Users/me/workspace/realm-development-tools/rebuild-sync-app/temp",
  "url": "https://realm.mongodb.com/groups/6258............../apps/625c............../dashboard"
}
Check out your app: cd .//Users/me/workspace/temp && realm-cli app describe
```

Check out the filepath. If the filepath is someplace unexpected, or if there is already a Realm App
configuration file there, the script will hang at this step. Delete the config files or specify
a different location where ln 12 of the script passes `--local temp`.

If you change the directory for the temp files generated by app creation, make sure to update
ln 13 to the correct location so it can automatically delete the files afterward.

### Did the script create a new Realm App, but fail to push the updated config files?

Make sure you provided the correct path to your existing Realm App config files in an
environment variable called `$REALM_APP`. Alternately, make sure you source your .bashrc 
or otherwise make the environment variable available to the script.

If the script starts to update the config, but then hangs, it might be prompting you to 
accept the changes. Try running the `realm-cli push` update command from ln 15 manually.
You'll need to provide the new APP ID and the existing Realm App config directory when you do.

Alternately, this step could fail if a Realm App was not created successfully and there is no
`$NEW_APP_ID`. Check the Realm dashboard in MongoDB Atlas to see if a new app was created, or
if there are error messages.
