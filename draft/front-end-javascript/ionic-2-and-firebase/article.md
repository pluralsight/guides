# Ionic 2 and Firebase Part 1

In this guide I will walk you though creating a data service using ionic 2 and Googles Firebase realtime database.

---

## Creating the project

First make sure the ionic cli is installed and up to date. 
```
$ npm install -g ionic@beta 
$ npm install -g cordova 
```
Next lets generate our project. We will be making a simple to do list.

```
$ ionic start todoer blank --v2 --ts
```
Lets take a look at the command. 
* ionic start 
 * creates a new project
* todoer
 * the name of our project
* blank
 * the template we want to use. We are starting a blank project
* --v2
 * Use ionic 2. Since ionic 2 is still in beta start will default to version 1
* --ts
 * Use typescript. Both angular 2 and ionic 2 are writen in typescript, and it appears to be the defacto going forward. So, we will use it too. 

```
$ cd todoer
```

## Setting up firebase
Google recently purchased firebase and has changed the way things are setup so if you already have a firebase account you can skip to the next section. If not lets set one up now.

go to google.firebase.com and click sign up.


