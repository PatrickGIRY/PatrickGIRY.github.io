# Application avec React JS / Spring boot / fitness

## Front ReactJS

### Structure projet

    /src/main/resources
    +- static
    application.yml

### Exclure les sources statiques

      <resources>
        <resource>
          <directory>src/main/resources</directory>
          <excludes>
            <exclude>**/static/node_modules/**</exclude>
            <exclude>**/static/src/**</exclude>
            <exclude>**/static/package.json</exclude>
            <exclude>**/static/index.jsx</exclude>
          </excludes>
        </resource>
      </resources>

### Plugin Front

[https://github.com/eirslett/frontend-maven-plugin](https://github.com/eirslett/frontend-maven-plugin)

### package.json

    "scripts": {
      "start": "webpack-dev-server --progress --colors --config ./webpack.config.js",
      "bundle": "webpack --config ./webpack.config.js"
    }

    "babel": {
      "presets": {
          "env",
          "react",
          "stage-2" /*??*/
      }
    }

    npm init

    npm install babel-core -D
    npm install babel-eslint -D
    npm install babel-loader -D
    npm install babel-preset-stage-2 -D ???
    npm install css-loader -D
    npm install eslint-config-airbnb -D
    npm install eslint-config-prettier -D
    npm install eslint-plugin-import -D
    npm install eslint-plugin-jsx-a11y -D
    npm install eslint-plugin-prettier -D
    npm install eslint-plugin-react -D
    npm install html-webpack-harddisk-plugin -D
    npm install prettier -D
    npm install react-hot-loader -D
    npm install style-loader -D
    npm install url-loader -D
    npm install webpack -D
    npm install webpack-dev-server -D

    npm install babel-prest-env -S
    npm install bootstrap -S
    npm install eslint -S
    npm install history -S
    npm install html-webpack-plugin -S
    npm install material-ui -S
    npm install material-ui-icons -S
    npm install react -S
    npm install react-bootstrap -S
    npm install react-dom -S
    npm install react-intl -S
    npm install react-redux -S
    npm install react-router -S
    npm install react-router-dom -S
    npm install react-router-redux -S
    npm install reactstrap -S
    npm install redux -S
    
