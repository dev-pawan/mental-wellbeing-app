# Content Management System for Generating Behavioural Apps
Pawan Sonawane
Department of Computer and Information Sciences, University Of Strathclyde, Glasgow, Scotland

## Overview
This project involved the creation of a Content Management System to generation behaviour change applications based on [volitional help sheets](https://pubmed.ncbi.nlm.nih.gov/20309777/) and [implementation intentions](https://en.wikipedia.org/wiki/Implementation_intention). The software was developed using the Next.js framework with TailwindCSS fot the front-end PHP and MySQL for the back-end and database solution, and Auth0 for user security and authentication.

## Base installation
The project is currently being hosted on [Vercel](https://cs546.vercel.app/). In order to run the software locally, please make sure to have [Node.js](https://nodejs.org/en), [NPM](https://www.npmjs.com/) and [Yarn](https://yarnpkg.com/) installed. Afterwards, run the following command in your terminal, making sure you are in the root directory of the folder:
```console
yarn dev
```
The CMS will consequently run on [`localhost:3000`](http://localhost:3000/). For more information, please refer to the official [Next.js documentation](https://nextjs.org/docs).

## Core CMS

The CMS has a modular structure. This means that a page in the CMS inside the folder `pages/*` mostly consists of assets, functions and components from the folders `components/*` and `features/*`. This approach was adopted because some features are shared by both the CMS and the output application, and because it enables the files in `pages/` to have dynamic URLs in the final application, which offers additional features such as bookmarking projects and enhanced security by matching URLs to the project IDs of the current user.

Be mindful of what users can see - in the project, route protection ensures that users cannot access (and therefore modify, etc.) projects that are not theirs within the editor. This is achieved by using Auth0’s `user` feature, and comparing it with the database. These are further explained in the Authorisation & Security and Backend & Database sections.

### Editor

The editor has 4 sections, each in the features folder:

- Rules Editor → `features/editor/rules`
- Review Questions Editor → `features/editor/reviews`
- Styling → `features/editor/design`
- Finish & Deploy → `features/editor/launch`

The main editor page `pages/editor/[projectID].tsx` has the layout that displays the above features in the side bar, along with other navigation elements such as a link to the user’s dashboard and a pop-up modal for more info about a page. 

In general, it is recommended to use or create new `components` and then import them into a `page`.

### Other pages

The CMS has two other main pages besides the editor: the dashboard and the home page. The home page provides an overview of the project and its goals, and should be updated regularly to reflect the current status of the project.

The dashboard page shows the list of projects and their details, and should only be changed if there are any significant modifications to the projects themselves.

### Components

Most components are designed to be as versatile as possible, since they are used in various parts of the project. Most components do not have any mandatory attributes, but rather use default values if they are not specified. For instance, the custom `Button` component can work without any of its attributes, but it will use predefined ones if needed.

It is also a good practice to allow interactable components to have their `onClick()` (or similar) function defined later. This is in line with the previous guideline of making components as general as possible.

If a component cannot be fully generalized, it is better to group components by their purpose. In the current project, this has been done with components related to forms.

## End application

The output application is a **webapp that responds to different devices and gets its data from a database.** This means that it can be used by any device that has a web browser that is up to date.

Most of the things that we talked about in the Core CMS section are also true for the output application because they are made of the same parts. So, we have to be careful that when we change something in the CMS, it does not affect the output application in a bad way and the other way around.

### Output application

The output application is mostly in the `pages/application/[projectId].tsx` and `features/application/*` files.

Like the core CMS, the output application uses the square brackets to make the web address change depending on the project. This makes it easy to go to a specific project directly.

### Possible future implementations

Because the output application gets its data from a database, we can use any way of showing the data that can connect to the database. For example, we can make apps for different devices that show the data and put them on the app store. The way of doing this would be very similar to the way we do it for the output application now.

## Authorisation & security

Use different words to express the same meaning in the sections below. Keep the markdown formatting and the headings.
"The CMS relies on Auth0 for login management. Auth0 offers a secure service and lets users sign in with either an email and password or SSO with various accounts, of which this project supports Google SSO. Both the main CMS application and the backend database depend on Auth0 being part of the project.

### Setup

At least one system administrator needs an Auth0 account. All information required for the below implementation can be found on this page.

To configure the Auth0 project for the application to work with it, go to Applications → CS546 CMS Project → Settings, and fill in the following information:

- Allowed Callback URLs
- Allowed Logout URLs
- Allowed Web Origins
- Allow Cross-Origin Authentication → ON
- Allowed Origins (CORS)

Each of these should have the corresponding URL for the end application (and in the case of the non-CORS options, any test domains such as `localhost:3000` or test vercel deployments)

### Implementation

To add Auth0 functionality, run `yarn add auth0` in the terminal to install/update it in the project."

> **Note**: These variables are highly sensitive and must be kept securely out of public access.
- `AUTH0_SECRET`
- `AUTH0_BASE_URL`
- `AUTH0_ISSUER_BASE_URL`
- `AUTH0_CLIENT_ID`
- `AUTH0_CLIENT_SECRET`
- `AUTH0_DOMAIN`
> **Note:** These variable won't always be the same accross versions or deplouments. For example, `AUTH0_BASE_URL` will be different for the test deployment and the production deployment.
> **Note**: If continuing to host on Vercel, ensure that the above environment variables are set in the Vercel project settings.

The following sections contain sentences that have been rephrased using different words. The markdown formatting and the headings have been preserved.

The application's current version combines the Auth0 user database with a separate database for the projects. The `project_joins` table connects these databases, and uses the user's email as the primary key. The Backend & Database section provides more details on this.

For Next.js applications to work with Auth0, the `<UserProvider>` tag must enclose the `<Component {...pageProps}/>` element in the `_app.tsx` file. This tag also needs to have the `loginUrl` and `profileUrl` attributes set to the corresponding URL paths. In this project, these are:

`loginUrl="/api/auth/login"`

`profileUrl="/api/auth/me"`

### Notes:

- This project supports email & password and Google sign ins by default, but more options can be added from the Auth0 project management page.
- Make sure that the `images` configuration in the `next.config.js` file allows for the image domains of any added login providers.
- The `pages/api/auth` folder contains custom API endpoints for vercel compatibility. These files are necessary if hosting on vercel. If using a different host, these might be replaced by the auto-generated versions created by the `[...autho].ts` file - check with the hosting provider beforehand.

## Backend & database

The backend consists of a series of PHP files that access the database from the frontend. Each file has an SQL statement that runs a query on the database when it is invoked. This was done to enable remote connection to the university MySQL database.

### Database Structure

The database has 2 main tables with an additional table to facilitate many-to-many relationships between them. See the ERD for the complete structure.

#### Projects Table

The projects table stores all the data of each project. The pID column is the primary key, used to link projects to their owners in the `project_joins` table, and is set to auto-increment.

#### Project Joins Table

This is used as a composite key to associate projects with owners. This table simply contains a pID item and an owner’s email.

#### Users Table

The current version of the project does not fully utilize this table, and instead gets the information required from Auth0. In this case, the user’s email address is considered their unique identifier and acts as a primary key for the user in the `project_joins` table.

If choosing to use a different login provider or make a custom one within this database, this Users table can store the user’s email as well as any other required information associated with a user, unrelated to projects. The table should be updated to include any additional information required that is not present in the current version.

> **Note**: Remember when creating a custom login to store passwords and other sensitive data according to best practices, such as adding salts and encryption.

### Backend & connectivity

The following sections contain sentences that need to be rewritten. Keep the markdown formatting and the headings:

#### PHP/Database to CMS

The file `api/queryDatabase.ts` stores the queries for a specific function in the front end. Each function communicates with the database by calling a matching PHP file on the university servers.

To make a new query:

- Make a new PHP file. You can use the template file as a guide.
- Include the following:
    - Inputs from CMS (Optional)
    - SQL statement
    - Further processing
    - Returned data
- Include a matching function in the `queryDatabase.ts` file

You can call this function from any page where it is needed after importing it. You can access any returned data using `await res.json()`, where `res` is the result of `await fetch(PHP_URL, REQUEST_PARAMETERS)`

### Notes

- Check that the environment variable `NEXT_PUBLIC_PHP_URL` is set up correctly.
    - This lets you call PHP fetches from a single URL, no matter where you are in the code.
    - This is defined in the `next.config.js` file.
- Check that when you send information to PHP, the variables in the JSON object passed in have the same names inside the PHP. An example below shows that the information defined in the Next.js code has the same name as in the `$array->VARIABLE` part of the PHP code.

```ts
const res = {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify({
        ifBlock: JSON.stringify(newIfThenBlock),
        thenBlock: JSON.stringify(newThenBlock),
        ID: projectId,
    }), 
};
```
*Next.js code-snippet*

```php
$json = file_get_contents('php://input');
$array = json_decode($json);
$ifBlock = $array->ifBlock;
$thenBlock = $array->thenBlock;
$projectID = $array->ID;
```
*PHP file*
