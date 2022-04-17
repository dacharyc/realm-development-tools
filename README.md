# Realm App Development Tools

This is an UNOFFICIAL repository containing tools to help develop
applications that use [Realm Database](https://realm.io).

Over time, I'm planning to add helpful scripts or code examples that I use
while developing Realm apps. I welcome PRs or suggestions for tools that
would be helpful to Realm developers.

## The Tools

- [Rebuild Sync App](/rebuild-sync-app/): This is a script to automate the 
  process of tearing down an existing [Realm Sync](https://www.mongodb.com/realm/mobile/sync) 
  app, dropping the MongoDB Atlas DB that stores the data, and creating 
  a new Realm App with the same app configuration. Use when 
  developing a Realm Sync app and iterating on the schema - 
  i.e. making breaking schema changes.
