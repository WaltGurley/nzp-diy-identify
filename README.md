# nzp-diy-identify

> A Vue project to build an image identification web application using locally hosted images and tabular data for each image. This version was created specifically for an exhibit at the National Zoological Park.

## Development Setup

1. Install dependencies and start local server

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
```

2. Ensure proper image and data setup for application
  * Data must be stored as a .csv file named `imageInfo.csv` in the directory `/static/`
  * Images must be stored in the directory `/static/images/`. The names of each image must exactly match the corresponding name in the 'ImageName' column of `imageInfo.csv`. This includes extension capitalization (e.g., .jpg vs .JPG). As an example, to quickly convert '.JPG' file extensions to lowercase run the following bash script from the command line in the directory `/static/images/`:
  ```bash
  bash for file in *.JPG; do mv "$file" "${file/.JPG/.jpg}"; done
  ```

## Build production application

``` bash
# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```
**Note for this build:**

Changed assets path in ```/config/index.js``` for correct reference to resources on GitHub pages:

```js
assetsPublicPath: '/' --> assetsPublicPath: './'
```


## Additional modules added for this project
``` bash
# install modules to use SASS
npm install sass-loader node-sass style-loader --save-dev

# install csv-loader for easy loading of local csv files
npm install csv-loader --save

# install include-media for easy media queries in SASS
npm install include-media --save

# install zooming to handle image zooming
npm install zooming --save

# install vue-awesome, a Font Awesome component for Vue.js
npm install vue-awesome --save
```


For detailed explanation on how things work, checkout the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
