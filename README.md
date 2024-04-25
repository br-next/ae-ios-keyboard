# ae-ios-keyboard
This is a smart After Effects Template for visualising a text input being written by an iOS Keyboard (German). 


![Screen Recording 2024-04-25 at 15 39 53_2](https://github.com/br-next/ae-ios-keyboard/assets/102033283/4eb1660c-ca7d-496f-838a-ab22ae9d059b)



### Please note
This template is currently embedded in a iOS-Twitter Layout. Feel free to customize the Interface as you like. You can also copy-and-paste the keyboard into other interfaces by including the comp "iOS_Keyboard pre" and the layers "Cursor", "Texteingabe" and "CTRL". 
The template doesn't include any word suggestions as implemented in a true iOS keyboard for complexity reasons. 

### Functionality
Text-input is possible via the the layer "TEXTEINGABE" by simply typing in the desired text. Upper- and lowercase letters are represented in the keyboard's layout while punctuation is skipped during playback due to the missing second layer of the keyboard (this might be added in future versions). Instead the punctuation is faked by visualising a keystroke of the represented symbol at the last pressed letter. 
The typespeed can be adjusted in the "CTRL"-Layer: The slider "speed" represents the time to completion of the animation in seconds.
The template is based on shape- and text-layers and can be indefinitely scaled up. 

# How it works
The template is driven by the "TEXTEINGABE"-Layer in the main comp, while the comp "iOS_Keyboard pre" contains all the needed layers and expression for the keyboard animation. 

### Animation: Key Stroke
![pressedKey](https://github.com/br-next/ae-ios-keyboard/assets/102033283/a84cf447-c6f2-4b08-935b-ed1f5b463ae2)
*the pressedKey-shape*


The comp "iOS_Keyboard pre" displays the actual keyboard as shape- and text layers. The comp "pressedKey" represents the keystroke animation, which is driven by the 'position'-property:

```Javascript

input = comp("240423 - iOS Keyboard Twitter").layer("TEXTEINGABE").text.sourceText.toLowerCase(); // Convert string to lowercase
inArr = input.replace(/[^\s\w]/g, ""); //Remove all punctuation marks since they are not represented in this version of the keyboard.
inArrS = inArr.split(""); //Split input String into array
inL = inArrS.length; //Get length of array
pos = [];

// Get Position of all pressed chars and put it into pos[]
for (var i = 0; i < inL; i++) {
    var currentLayer = thisComp.layer(inArrS[i]);
    if (currentLayer) {
        pos[i] = currentLayer.transform.position;
    } else {
        pos[i] = [0, 0]; // If the layer doesn't exist, set a default position
    }
}
```


We now have an array filled with the position properties of all represented text-layers in the text input. Now let's make sure that we don't have any faulty values in our array: 

```Javascript
// Check if pos array is not empty
if (pos.length > 0) {
    var index = Math.floor(time);

    // Check if the index is within the bounds of the pos array
    if (index >= 0 && index < pos.length) {
        posAtTime = pos[index]; // Set posAtTime to the position at the index
    } else if (index >= pos.length) {
        posAtTime = pos[pos.length - 1]; // Set posAtTime to the last known position
    }
}
```

By using the `Math.floor`-function we can make sure to only change values every full second. 
Now, let's animate using the linear interpolation: 

```Javascript
// Use the linear() function to interpolate between positions
linear(time, 0, inL - 1, posAtTime, posAtTime);
```

Simultaneously the highlighted letter inside the shape of "pressedKey" changes to the value of the array:

```Javascript
input = comp("240423 - iOS Keyboard Twitter").layer("TEXTEINGABE").text.sourceText; 
inArr = input.split(""); //Split input String into array
t = (time < input.length) ? inArr[Math.floor(time)] : inArr[(input.length)-1];

t;
```

Since a keystroke visualisation on iOS keyboards only last for about a fraction of a second we have to make sure the "pressedKey"-comp is visible only during a short period of time by using the opacity-property:
This property is keyframed and looped in an expression: 

```Javascript
input = comp("240423 - iOS Keyboard Twitter").layer("TEXTEINGABE").text.sourceText; // Input Text
inArr = input.split(""); //Split input String into array
inL = inArr.length; //Get length of array
inSpace = [];

for (i = 0; i < inL; i++){ //Check if there is the use of the space key in input
	inSpace[i] = inArr[i].match(/\s/) !== null; 
}

//If the space key is pressed set the opacity to 0
(time == 0 || inSpace[Math.floor(time)] == true) ? 0 : loopOut();
```

This expression also makes sure that the "pressedKey"-composition is not visible when a spacebar keystroke is visualised. 

### Switching between uppercase and lowercase

Switching between uppercase and lowercase is a fairly simple process, since we work with native textlayers on all visualised keys. The "CTRL"-layer in the "iOS_Keyboard pre" composition controls the switch between upper- and lowercase letters:

```Javascript
input = comp("240423 - iOS Keyboard Twitter").layer("TEXTEINGABE").text.sourceText;
inArr = input.split(""); //Split input String into array
inL = inArr.length; //Get length of array
var upperCase = []; 

for (i = 0; i < inL; i++){ 
	upperCase[i] = inArr[i].match(/[A-Z]/) !== null; //Create a boolean-array with true = uppercase and false = lowercase
}
if (time >= 0 && time < inL) {
	upperCase[Math.floor(time)]; 
}

else {
	value; 
}
```

This results in an true/false animation for every full second. Every visualised key accesses this property and changes to lower- or uppercase respectively: 

```Javascript
caps = thisComp.layer("CTRL").effect("CAPS")(1);

(caps == true) ? text.sourceText.toUpperCase() : text.sourceText; 
```

