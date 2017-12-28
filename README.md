# Vue IPL

> A web app to show IPL stats built on VueJS

Wait for data to load as it reads from CSV file and processes in chunks.
Once the data has been loaded and mapped to the views , it can be accessed again with the help of localStorage.

**Below are some useful points related to the project**
1. VueJS is used to build the component based system.
2. Vue-router is used for routing
3. Vue-material is used to give it a material design feel.
4. Highcharts library is used for the representation of data in form of charts.
4. App is responsive and can be used in any mobile devices.
5. Load time is reduced, through the introduction of Webpack and inbuilt bundler and minifier.
6. Papa parser is used to convert the CSV data to JSON and read it in chunks.
7. Offline usability is not introduced but utilized the localStorage of browser to reduce calling data reading and processing from csv file.

___
> [Here is the link to preview](https://abhishekbadola-ipl.netlify.com)
