# Dither-Snake
## Dark version of the snake game with shading and dithering
Made mostly to test how this type of shading would work. Only the snake head and food emit light, everything else is hidden in darkness. Stylish [Bayer4x4](https://en.wikipedia.org/wiki/Ordered_dithering) dithering on top of that, making the game canvas 1 bit.  
I recommend playing on a desktop device in full screen (F11), although I added support for mobile devices but it's very eh...  

![image](https://user-images.githubusercontent.com/82185066/198695161-e200febb-02c5-49f9-a410-c76ddc2aedf9.png)  

Play field size will change dynamically to the size of your screen, meaning if you want to make the play field bigger just press ctrl and scroll with mouse wheel.  

Food texture atlas:  
![food_atlas](https://user-images.githubusercontent.com/82185066/198753312-94a74d95-3cbd-4967-a43f-a6e54e40bc92.png)

Amazing particle animation:  
![2022-10-29 03-03-57](https://user-images.githubusercontent.com/82185066/198753324-6aaa2435-013d-4daa-a946-18e0a0c3f28c.gif)

SFX generated using [sfxr](https://sfxr.me/).

My high score)  
![image](https://user-images.githubusercontent.com/82185066/198754156-3d1f44fb-425e-4644-9cb8-fa7be5a31125.png)

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
