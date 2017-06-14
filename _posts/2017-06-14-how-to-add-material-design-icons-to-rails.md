---
title: How to add Material Design Icons to a Rails project
---

No blabbering about why you need material icons, or what they are. I assume if you are reading this you already know what material icons are, and you need them, for your Rails project. So here we go!

### Get the Icons
`npm install material-design-icons --save`

If you are using Rails 5.1, you can also use `yarn` to install the repo. [Rails now natively ships with yarn](http://weblog.rubyonrails.org/2017/4/27/Rails-5-1-final/).

`bin/yarn add material-design-icons`

### Add Node Modules to your asset path

You can skip this if you are using Rails 5.1.

Find this file in your project: `config/initializers/assets.rb`, and add the below piece of code to it. This will autoload node modules that you add.

```ruby
Rails.application.config.assets.paths << Rails.root.join('node_modules')
```

### Define the font face

A good idea is to create a new file say `icons.scss`, and then require it in `application.css.scss`. On a sidenote, I usually don't prefer to use the `require` statements, instead I prefer manually importing files using native `@import` for SCSS, this allows me to define global variables that I can use across files (maybe this is worth a separate post, but who knows.)

```css
@font-face {
  font-family: 'Material Icons';
  font-style: normal;
  font-weight: 400;
  src: url(/assets/material-design-icons/iconfont/MaterialIcons-Regular.eot); /* For IE6-8 */
  src: local('Material Icons'),
    local('MaterialIcons-Regular'),
    url(/assets/material-design-icons/iconfont/MaterialIcons-Regular.woff2) format('woff2'),
    url(/assets/material-design-icons/iconfont/MaterialIcons-Regular.woff) format('woff'),
    url(/assets/material-design-icons/iconfont/MaterialIcons-Regular.ttf) format('truetype');
}

.material-icons {
  font-family: 'Material Icons';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;  /* Preferred icon size */
  display: inline-block;
  line-height: 1;
  text-transform: none;
  letter-spacing: normal;
  word-wrap: normal;
  white-space: nowrap;
  direction: ltr;
  vertical-align: middle;

  /* Support for all WebKit browsers. */
  -webkit-font-smoothing: antialiased;
  /* Support for Safari and Chrome. */
  text-rendering: optimizeLegibility;

  /* Support for Firefox. */
  -moz-osx-font-smoothing: grayscale;

  /* Support for IE. */
  font-feature-settings: 'liga';
}

/* Rules for sizing the icon. */
.material-icons.md-18 { font-size: 18px; }
.material-icons.md-24 { font-size: 24px; }
.material-icons.md-36 { font-size: 36px; }
.material-icons.md-48 { font-size: 48px; }

/* Rules for using icons as black on a light background. */
.material-icons.md-dark { color: rgba(0, 0, 0, 0.54); }
.material-icons.md-dark.md-inactive { color: rgba(0, 0, 0, 0.26); }

/* Rules for using icons as white on a dark background. */
.material-icons.md-light { color: rgba(255, 255, 255, 1); }
.material-icons.md-light.md-inactive { color: rgba(255, 255, 255, 0.3); }

.material-icons.md-soft { color: rgba(255, 255, 255, 0.5); }
```

### Precompile your assets to use locally

`rails assets:precompile`

Voila! You have the icons. Now use them.

```
<i class="material-icons md-24 md-soft>question_answer</i>
```

If you are just trying out material design icons, and you don't yet want to work so hard to get them working for your project, you can just link them, simple and easy!

```
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

Thanks!
