# VueJS / MagicLink integration

![](https://media0.giphy.com/media/9r75ILTJtiDACKOKoY/giphy.gif?cid=ecf05e470vlnfa97n8igtg1fhs45te0hfr9g0rkdf6tzuqce&rid=giphy.gif&ct=g)

This template wants to test out the functionality of Magic Link (https://magic.link/), used with Ethereum (Rinkeby).
Any other EVM network is compatible (like Polygon) and you can use it to easily onboard your users in your app with their e-mail.
Approach is not 100% "crypto" but there are use cases where this strategy can be interesting.

Magic happens at `components/MagicLink.vue` where you can see you'll need a properly configured `.env` file like this:

```
VUE_APP_MAGIC_PUB_KEY=
VUE_APP_CONTRACT_ADDRESS=
VUE_APP_NETWORK=
```

After everything is setted up you can run VueJS project, you can find a live example here: https://turinglabsorg.github.io/magiclink-vue-boilerplate/

## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```
