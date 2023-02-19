# webpack-guide

## Гайд по сборке многостраничного сайта + линтеры (pug, scss, babel, eslint, stylelint, prittier)

### Установим все необходимые пакеты:

> webpack:

npm i --save -D webpack webpack-cli webpack-dev-server cross-env

pug:

npm i --save -D pug simple-pug-loader simple-pug-loader html-webpack-plugin

scss:

npm i --save -D sass sass-loader postcss postcss-loader postcss-preset-env css-loader autoprefixer resolve-url-loader mini-css-extract-plugin

babel:

npm i --save -D @babel/core @babel/preset-env babel-loader babel-eslint

eslint:

npm i --save -D eslint eslint-config-airbnb eslint-config-airbnb-base eslint-plugin-import

stylelint:

npm i --save -D stylelint stylelint-config-airbnb stylelint-config-rational-order stylelint-declaration-block-no-ignored-properties stylelint-order stylelint-scss

prettier:

npm i --save -D prettier eslint-config-prettier eslint-plugin-prettier prettier-eslint stylelint-config-prettier stylelint-prettier

Структура файлов

src / -- корневая папка
pages/
page-name/ -- папки с файлами для каждой отдельной страницы(page-name меняем на наше нужное название)
page-name.pug
page-name.scss
page-name.js
styles/
-- файлы со шрифтами, переменными и минимальными глобальными стилями
components/
component-name/ -- папки компонентов (component-name меняем на наше нужно название)

Конфиги:
в корне проекта создаем файлы .eslintignore, .eslintrc, .gitignore, .prettierrc, .stylelintrc, webpack.config.js

.gitignore:

node_modules
dist

.eslintignore:

webpack.config.js

webpack.config.js :

const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const fs = require('fs');

let mode = 'development';
if (process.env.NODE_ENV == 'production') {
mode = 'production';
}
console.log(mode + ' mode');

// тут получаем пути до js файлов и создаем entryPoints для каждой страницы
const PAGES_DIR = path.resolve(**dirname, 'src/pages');
const PAGES = fs.readdirSync(path.resolve(**dirname, 'src/pages'));
const entryPoints = Object.assign(
{},
...PAGES.map((page) => ({
[page]: `${PAGES_DIR}/${page}/${page}.js`,
}))
);

module.exports = {
mode: mode,
context: path.resolve(**dirname, 'src'),
entry: entryPoints,
output: {
filename: '[name].[contenthash].js',
path: path.resolve(**dirname, 'dist'),
clean: true,
},
devServer: {
historyApiFallback: true,
static: {
directory: path.resolve(\_\_dirname, 'dist'),
},
// тут нужно вместо index.html указать явно какую страницу открвыть напрмер page.html
open: '/index.html',
hot: false,
},
devtool: 'source-map',
plugins: [
new MiniCssExtractPlugin({
filename: '[name].[contenthash].css',
}),
// тут каждую страницу передаем в HtmlWebpackPlugin
...PAGES.map(
(page) =>
new HtmlWebpackPlugin({
filename: `${page}.html`,
template: `${PAGES_DIR}/${page}/${page}.pug`,
chunks: [page],
inject: 'body',
})
),
],
module: {
rules: [
{
test: /\.(sa|sc|c)ss$/,
      use: [
        MiniCssExtractPlugin.loader,
        'css-loader',
        {
          loader: 'postcss-loader',
          options: {
            postcssOptions: {
              plugins: [
                [
                  'autoprefixer',
                  {
                    overrideBrowserslist: ['> 1%', 'last 2 versions'],
                  },
                  'postcss-preset-env',
                ],
              ],
            },
          },
        },
        'resolve-url-loader',
        'sass-loader',
      ],
    },
    {
      test: /\.(png|svg|jpg|jpeg|gif)$/i,
type: 'asset/resource',
generator: {
filename: 'assets/images/[name][ext]',
},
},
{
test: /\.ico/i,
type: 'asset/resource',
generator: {
filename: 'assets/favicon/[name][ext]',
},
},
{
test: /\.(woff|woff2|eot|ttf|otf)$/i,
      type: 'asset/resource',
      generator: {
        filename: 'assets/fonts/[name][ext]',
      },
    },
    {
      test: /\.pug$/,
use: [
{
loader: 'simple-pug-loader',
options: {
pretty: true,
},
},
],
},
{
test: /\.m?js$/,
exclude: /node_modules/,
use: {
loader: 'babel-loader',
options: {
presets: ['@babel/preset-env'],
},
},
},
],
},
};

.eslintrc:

{
"parserOptions": {
"parser": "babel-eslint",
"sourceType": "module",
"allowImportExportEverywhere": true,
"ecmaVersion": 2015
},
"env": {
"browser": true,
"es6": true,
"jquery": true,
"node": true
},
"extends": [
"airbnb-base",
"plugin:prettier/recommended",
"prettier"
],
"globals": {
"$": "readonly",
"jQuery": "readonly"
},
"plugins": ["prettier"],
"rules": {
"no-new": "off",
"import/no-unresolved": "off",
"prefer-regex-literals": "off",
"no-underscore-dangle": ["off", { "allow": true }],
"prettier/prettier": "error"
}
}

.stylelintrc:

{
"extends":[
"stylelint-config-rational-order",
"stylelint-prettier/recommended"
],
"plugins": [
"stylelint-scss",
"stylelint-order",
"stylelint-declaration-block-no-ignored-properties",
"stylelint-config-rational-order/plugin",
"stylelint-prettier"
],
"rules": {
"prettier/prettier": [true],
"order/properties-order": [],
"plugin/rational-order": [true, {
"border-in-box-model": false,
"empty-line-between-groups": false
}],
"max-nesting-depth": 3,
"declaration-property-value-blacklist": {},
"rule-empty-line-before": [
"always-multi-line",
{
"except": [
"first-nested"
],
"ignore": [
"after-comment"
]
}
],
"order/order": [
"custom-properties",
"dollar-variables",
"declarations",
"rules",
"at-rules",
{
"type": "at-rule",
"name": "media"
}
]
}
}

.prettierrc.json:

{
"singleQuote": true,
"arrowParens": "always",
"max-len": ["error", 80, 2],
"printWidth": 80,
"tabWidth": 2,
"useTabs": false,
"endOfLine": "auto",
"overrides": [
{
"files": "\*.pug",
"options": {
"parser": "pug",
"singleQuote": true,
"max-len": ["error", 80, 2],
"printWidth": 80
}
}
]
}

##Scripts: ну и добависм скрипты для запуска webpack и linter-ов в package.json:

"scripts": {
"start": "cross-env NODE_ENV=development webpack serve",
"dev": "cross-env NODE_ENV=development webpack",
"build": "cross-env NODE_ENV=production webpack",
"deploy": "gh-pages -d dist",
"lint": "eslint src",
"lint:fix": "eslint src --fix",
"stylelint": "stylelint \"src/**/\*.scss\"",
"stylelint:fix": "stylelint \"src/**/\*.scss\" --fix"
}

Но лучше всего в файл package.json , стерев весь предыдущий код, вставляем следующий код, где в первой строке вместо name - пишем своё название, а в конце в трёх строках вместо TestSlider пишем своё название репозитория:

{
"name": "testslider",
"version": "1.0.0",
"description": "",
"main": "index.js",
"scripts": {
"start": "cross-env NODE_ENV=development webpack serve",
"dev": "cross-env NODE_ENV=development webpack",
"build": "cross-env NODE_ENV=production webpack",
"deploy": "gh-pages -d dist",
"lint": "eslint src",
"lint:fix": "eslint src --fix",
"stylelint": "stylelint \"src/**/\*.scss\"",
"stylelint:fix": "stylelint \"src/**/\*.scss\" --fix"
},
"dependencies": {},
"devDependencies": {
"@babel/core": "^7.19.1",
"@babel/preset-env": "^7.19.1",
"autoprefixer": "^10.4.11",
"babel-eslint": "^10.1.0",
"babel-loader": "^8.2.5",
"cross-env": "^7.0.3",
"css-loader": "^6.7.1",
"eslint": "^8.23.1",
"eslint-config-airbnb": "^19.0.4",
"eslint-config-airbnb-base": "^15.0.0",
"eslint-config-prettier": "^8.5.0",
"eslint-plugin-import": "^2.26.0",
"eslint-plugin-prettier": "^4.2.1",
"html-webpack-plugin": "^5.5.0",
"mini-css-extract-plugin": "^2.6.1",
"postcss": "^8.4.16",
"postcss-loader": "^7.0.1",
"postcss-preset-env": "^7.8.2",
"prettier": "^2.7.1",
"prettier-eslint": "^15.0.1",
"pug": "^3.0.2",
"resolve-url-loader": "^5.0.0",
"sass": "^1.54.9",
"sass-loader": "^13.0.2",
"simple-pug-loader": "^0.2.1",
"stylelint-config-prettier": "^9.0.3",
"stylelint-prettier": "^2.0.0",
"webpack": "^5.74.0",
"webpack-cli": "^4.10.0",
"webpack-dev-server": "^4.11.0"
},
"repository": {
"type": "git",
"url": "git+https://github.com/JuriJuriM/TestSlider.git"
},
"author": "",
"license": "ISC",
"bugs": {
"url": "https://github.com/JuriJuriM/TestSlider/issues"
},
"homepage": "https://github.com/JuriJuriM/TestSlider#readme"
}
