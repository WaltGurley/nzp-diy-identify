# nzp-diy-identify

> A Vue project to build an image identification web application using locally hosted images and tabular metadata for each image. This version was created specifically for an exhibit at the National Zoological Park

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

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
