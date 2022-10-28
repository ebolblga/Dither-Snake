# Dither-Snake
## Dark version of the snake game with shading and dithering
Made mostly to test how this type of shading would work. Only snake head and food emit light, everything else is hidden in darkness. Stylish [Bayer4x4](https://en.wikipedia.org/wiki/Ordered_dithering) dithering on top of that, making the game canvas 1 bit.  

![image](https://user-images.githubusercontent.com/82185066/198695161-e200febb-02c5-49f9-a410-c76ddc2aedf9.png)  

Food atlas:  
![image](https://user-images.githubusercontent.com/82185066/198698638-9498c4a2-36c1-4690-ad5d-471960ef7545.png)

SFX generated by [sfxr](https://sfxr.me/)

## How to launch yourself using [Node.js](https://nodejs.org/en/) and [Visual Studio Code](https://code.visualstudio.com/download)

```bash
# install yarn
npm install --global yarn

# now install all modules
yarn

# now start project
yarn dev

# build ssg version
yarn generate
```

## License
This program is licensed under the MIT License. Please read the License file to know about the usage terms and conditions.
