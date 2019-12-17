<template>
  <div id='app-container'>
    <div id='theme-container' :style='getCSSColorScheme()' :class='{ toggleOff: !this.$store.state.settings.enableThemeAnimations }'> </div>
    <div id='wrap' width='100vw' height='100vh' >
      <canvas id='output' width='2000px' height='2000px'></canvas>
    </div>
    <div id='app'>
      <div id='nav'>
        <router-link to='/home'>Now</router-link>
        <!-- <router-link to='/today'>Today</router-link> -->
        <router-link to='/about'>About</router-link>
      </div>
      <keep-alive>
        <router-view/>
      </keep-alive>
    </div>
    <div id='sources'>  
      <img id='base' crossorigin='anonymous' width= '2000' height= '2000' src='https://cdn.glitch.com/0f517177-828e-4f09-bd56-7413c9736e48%2FXtra.png?v=1576216285048' />
      <img id='xtra' crossorigin='anonymous' width= '2000' height= '2000' src='https://cdn.glitch.com/0f517177-828e-4f09-bd56-7413c9736e48%2Fxtra2.png?v=1576272681891' />
    </div>
  </div>
</template>

<script lang='ts'>
import { Component, Vue } from 'vue-property-decorator';
import { Themes } from './themes';
import { DateTime } from 'luxon';
import firebase from 'firebase';
import firebaseui from 'firebaseui';

@Component({})
export default class App extends Vue {
  getCurrentColorScheme() {
    return this.getColorSchemeFromId(this.$store.state.settings.colorTheme);
  }

  getColorSchemeFromId(themeId: string) {
    return Themes.filter((t) => t.id === themeId)[0] || Themes[0];
  }

  getCSSColorScheme() {
    let themeGradient;
    var currentColorScheme = this.getCurrentColorScheme();
    if (this.$store.state.settings.colorTheme === 'theme15') {
      var currentDate = DateTime.local().setZone('America/Los_Angeles');
      if (currentDate.hour >= 21 && currentDate.hour <= 4) {
        themeGradient = this.getColorSchemeFromId('theme12');
      } else if (currentDate.hour <= 9) {
        themeGradient = this.getColorSchemeFromId('theme14');
      } else if (currentDate.hour <= 11) {
        themeGradient = this.getColorSchemeFromId('theme4');
      } else if (currentDate.hour <= 15) {
        themeGradient = this.getColorSchemeFromId('theme6');
        console.log('Theme 6');
      } else if (currentDate.hour <= 17) {
        themeGradient = this.getColorSchemeFromId('theme7');
      } else {
        themeGradient = this.getColorSchemeFromId('theme1');
      }
    }
    else if (this.$store.state.settings.colorTheme === 'theme17') {
        document.getElementById('wrap').style.display = 'inline';
        console.log('Theme 17');
    }
    else {
      themeGradient = currentColorScheme;
      // document.getElementById('wrap').style.display = 'none';
    }
    return {
      '--gradient-colors': themeGradient.gradientColors.join(', '),
      '--button-menu-color': themeGradient.btnMenuColor,
      '--button-submenu-color': themeGradient.btnSubmenuColor,
      '--button-hover-color': themeGradient.btnHoverColor,
      '--gradient-count': themeGradient.gradientColors.length,
    };
  }
}

  // Your web app's Firebase configuration
let firebaseConfig = {
    apiKey: 'AIzaSyDWaQp0SxDc4t_aaUIxdDI_Zb0rf8YWASc',
    authDomain: 'helloyeet-c3c84.firebaseapp.com',
    databaseURL: 'https://helloyeet-c3c84.firebaseio.com',
    projectId: 'helloyeet-c3c84',
    storageBucket: 'helloyeet-c3c84.appspot.com',
    messagingSenderId: '969131500601',
    appId: '1:969131500601:web:f47d01d3ff7c4eef',
  };
  // Initialize Firebase
firebase.initializeApp(firebaseConfig);

// -----------------------------------------------BUbbles
var imgWidth = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
var imgHeight = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
// This is dead simple, just heavily-commented!

// Some varants to tweak.
var NUM_CIRCLES = 80;
var MIN_SIZE = 50;
var MAX_SIZE = 200;

// Returns a random int between two numbers.
function getRndInt(min, max) {
 
    return Math.floor(Math.random() * (max - min + 1)) + min;
 
}

// Cache refs to our canvas, context and images.
var c = document.getElementById('output');
var ctx = c.getContext('2d');
var sources = document.getElementById('sources');
var imgBase = document.getElementById('base');
var imgXtra = document.getElementById('xtra');

// Timestamp var used in the update-render loop.
let t = 0x0;

/**
 * An object 'class' to describe a circle which animates
 * its size (radius) and which can randomize its position.
 **/
var Circle = {
    x: 0,
    y: 0,
    size: 0,
    _needsRandomized: false,

    /**
     * Set a random position and max size
     **/
    randomize: function() {
        this.x = getRndInt(50, c.width - 50);
        this.y = getRndInt(50, c.height - 50);
        this.maxSize = getRndInt(MIN_SIZE, MAX_SIZE);
    },

    /**
     * Animates the size up and down via a sine calculation against a passed-in
     * timestamp factorial.  (See the main program update() method).
     * Accepts an offset so different instances will animate out-of-sync
     * (if ofs was 0 for all instances, they would synchronize).
     * When the circle is fully-shrunk, it randomizes its position and max size.
     **/
    update: function(t, ofs) {
        this.size = Math.abs(Math.round(Math.sin(t + ofs) * this.maxSize));

        if (this.size < 2) {
            if (this._needsRandomized) {
                this.randomize();
                this._needsRandomized = false;
            }
        } else {
            this._needsRandomized = true;
        }
    },

    /**
     * Draws a circle to the context at the current position and size.
     * NOTE: this doesn't open or close a path, or apply a fill or stroke.
     * It assumes a path has already been opened in the context.
     * (See main program render() method)
     **/
    draw: function() {
        ctx.moveTo(this.x, this.y);
        ctx.arc(this.x, this.y, this.size, 0, 2 * Math.PI);
    }
};

/**
 *
 * MAIN PROGRAM
 *
 **/

// Create multiple Circle instances in an array.
var circles = [];
for (var i = 0; i < NUM_CIRCLES; i++) {
    var circle = Object.create(Circle);
    circle.randomize();
    circles.push(circle);
}

/**
 * Tell each mask to update itself.  Pass in the current time
 * and an offset integer so the animations do not synchronize.
 **/
function update() {
    t = 0.001 * Date.now();
    circles.forEach(function(circle, idx) {
        circle.update(t, idx);
    });
}

/**
 * Main render method.
 * NOTE: we could have the Circle class drawing its own clip path and
 * image, but for performance reasons we do the main rendering here.
 * So we have one clip path created from multiple circles, instead of 
 * multiple clip paths (and image renders) per frame.
 **/
function render() {
    // Draw the background first.
    // Since this will cover everything from the last frame, 
    // there's no need for an explicit 'clear the canvas' step.
    ctx.drawImage(imgBase, 0, 0, imgWidth, imgHeight);

    // Save the state (before any clip regions exist),
    // and begin a new path.
    ctx.save();
    ctx.beginPath();

    // Draw each Circle instance, at its current position
    // and size, into the open path.
    // (Note that nothing gets visibly 'drawn' here - we're 
    // just defining the shape of the path.)
    circles.forEach(function(circle) {
        circle.draw();
    });

    // Close the path and flag it as a clipping region
    // for any subsequent drawing.
    ctx.closePath();
    ctx.clip();

    // Draw the 'I Am The Night' image, which will be clipped
    // by our path, so it is drawn 'into' the circles.
    ctx.drawImage(imgXtra, 0, 0, imgWidth, imgHeight);

    // Restore the pre-clip state, so that no further
    // clipping will occur.  Otherwise, the bg
    // would get clipped when we draw it next frame.
    ctx.restore();
}

// Main update-render loop.
function loop() {
    requestAnimationFrame(loop);
    update();
    render();
}

// Kick it off when our images are fully loaded.
imagesLoaded(sources, function() {
    console.log('Images Loaded');
    loop();
});

</script>


<style lang='scss'>

@keyframes AnimatedTheme {
  0% { background-position: 50% 0%; }
  50% { background-position: 51% 100%; }
  100% { background-position: 50% 0%; }
}

html, body, #app-container, #theme-container {
  height: auto;
  min-height: 100vh;
  width: 100%;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  overflow-x: hidden;
  overflow-y: scroll;
}

#theme-container {
  z-index: -10;
  position:fixed;
  top: 0px;
  left: 0px;
  bottom: auto;
  right: auto;
  overflow: hidden;

  background: linear-gradient(to bottom, var(--gradient-colors, '#42b983, #2f9768'));
  background-size: 400% 400%;
  animation: AnimatedTheme 20s ease infinite;

  &.toggleOff {
    background-size: 100%;
    animation: none;
  }
}

#wrap {
  z-index: -8;
  position:fixed;
  top: 0px;
  left: 0px;
  bottom: auto;
  right: auto;
  overflow: hidden;
  display: none;
}

#sources {
	display: none;
}

#app {
  font-family: 'Niramit', Avenir, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #fff;
  max-width: 600px;
  margin: 0 auto;
  padding-bottom: 40px;
}

#nav {
  padding: 30px;
  a {
    background-color: var(--button-menu-color, #2c3e50);
    display: inline-block;
    padding: 16px 30px;
    font-weight: bold;
    color: rgba(255, 255, 255, 0.6);
    border-radius: 4px;
    text-decoration: none;
    vertical-align: middle;
    margin: 8px;
    box-shadow: 0 4px 16px 8px rgba(0, 0, 0, .15);
    transition: 150ms ease;

    &:before {
      display: inline-block;
      margin-left: -10px;
      margin-right: 15px;
      content: '';
      height: 24px;
      width: 8px;
      border-radius: 4px;
      background-color: #1c1e20;
      vertical-align: middle;
      transition: 150ms ease;
    }

    &.router-link-active {
      color: #fff;

      &:before { background-color: #d5dee7; }
    }
  }
}


.sub-nav-item {
  background-color: transparent;
  border-radius: 2px;
  font-size: 14px;
  display: inline-block;
  text-decoration: none;
  color: #fff;
  text-transform: uppercase;
  font-weight: 700;
  margin: 0 5px;
  padding: 6px 16px;
  transition: 150ms ease;

  &:hover {
    background-color: var(--button-hover-color, rgba(#2f9768, .4));
  }

  &.router-link-exact-active {
    background-color: var(--button-submenu-color, #2f9768);
    box-shadow: 0 0 8px 4px rgba(100, 100, 100, .1);
  }
}
</style>
