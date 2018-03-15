# react-native-svg-src
Render an SVG Image from local file such as src. This lib is fork on react-native-svg-uri@1.2.3. But I will keep updating for this one.

Install library from `npm`

```bash
npm install react-native-svg-src --save
```
## Props

| Prop | Type | Default | Note |
|---|---|---|---|
| `src` | `String` | undefined | the content of SVG

## <a name="Usage">Usage</a>
Maybe you feel why the src is the content of svg instead of the path of static file

That's because this lib was fork on react-native-svg-uri@1.2.3. extends his everything includes his known bug with prop source:
> [ANDROID] There is a problem with static SVG file on Android,
>  Works OK in debug mode but fails to load the file in release mode.
>  At the moment the only workaround is to pass the svg content in the svgXmlData prop.

Don't worry. Here is the temp solution. I promise I will proffer easiest usage when I'm free.

I suppose you put all svg in one folder. Make a getSvg.js as fileReader, launch it once before you developing.
```
│
│─ getSvg.js
│
└─ SVGFOLDER ─
              │─ foo.svg
              │─ bar.svg
```
Here is the getSvg.js from jianshu.com
```javascript
//  getSvg.js
var fs = require('fs');
var path = require('path');
const svgDir = path.resolve(__dirname, './SVGFOLDER');

function readfile(filename) {
  return new Promise((resolve, reject) => {
    fs.readFile(path.join(svgDir, filename), 'utf8', function (err, data) {
      console.log(data.replace(/<\?xml.*?\?>|<\!--.*?-->|<!DOCTYPE.*?>/g, ''));
      if (err) reject(err);
      resolve({
        [filename.slice(0, filename.lastIndexOf('.'))]: data,
      });
    });
  });
}

function readSvgs() {
  return new Promise((resolve, reject) => {
    fs.readdir(svgDir, function (err, files) {
      if (err) reject(err);
      Promise.all(files.map(filename => readfile(filename)))
        .then(data => resolve(data))
        .catch(err => reject(err));
    });
  });
}

readSvgs().then(data => {
  let svgFile = 'export default ' + JSON.stringify(Object.assign.apply(this, data));
  fs.writeFile(path.resolve(__dirname, './svgs.js'), svgFile, function (err) {
    if (err) throw new Error(err);
  })
}).catch(err => {
  throw new Error(err);
});
```
After run it, your files tree should looks like below
```
│
│─ svgs.js
│─ getSvg.js
│
└─ SVGFOLDER ─
              │─ foo.svg
              │─ bar.svg
```
The svgs.js export an object<K,V>. The K means the svg file name, the V means the content of the svg.

OK. Let's see the example

```javascript
// Svg.js
import React, { Component } from 'react';
import SvgSrc from 'react-native-svg-src';
import svgs from './../img/svgs';

export default class Svg extends Component {
  render() {
    const { src } = this.props;
    let svgXmlData = svgs[this.props.src];

    if (!svgXmlData) {
      let err_msg = `no such icon which called "${this.props.src}"`;
      throw new Error(err_msg);
    }
    return (
      <SvgSrc src={svgXmlData} />
    )
  }
}
```
```javascript
// example.js
import React, { Component } from 'react';
import { View } from 'react-native';
import Svg from 'yourPath/Svg';

export default class example extends Component {
  render() {
    return (
      <View style={{ height: 100, width: 100}}>
        {/* foo means foo.svg under your SVGFOLDER */}
        <Svg src="foo" />
      </View>
    )
  }
}
```