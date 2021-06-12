---
label: UI Development
icon: package
order: 900
---
# UI Development

Once you have run the [roulette-ui](https://github.com/sakuracasino/roulette-ui) repository, it will open a dev server in `http://localhost:9000`.

### Webpack

We use [webpack](https://webpack.js.org/) for building the bundle. `webpack.commmon.js` contains most of the configuration.

`npm start` servers the express server
`npm run build` generates a production-ready build in the `dist/` folder

### web3React

We use [webReact](https://github.com/NoahZinsmeister/web3-react) for connecting with the blockchain through *Metamask*.
We also use [ethers](https://docs.ethers.io/v5/) for interacting with contracts and big numbers.

All the actions and data retrieval are done by the `NetworkHelper` class.

### Redux

We use [React](https://reactjs.org/) and [Redux](https://redux.js.org/) for application data management.

There currently two slices:

`betPoolSlice.ts`, which manages the data related to the current bets you're doing

`networkSlice.ts`, which manages data from the blockchain.

### Typescript

We also use [TypeScript](https://www.typescriptlang.org/) for all our *JavaScript* code. Common types are in the `src/types.d.ts` file.


### CircleCI

We use a [CircleCI](https://circleci.com/) configuration file for all the continuous integration tasks.

When you merge something in the `master`, it will update automatically under `appdev.sakura.casino` using a S3 bucket.
When changes are merged to the `production` branch, `app.sakura.casino` will be deployed with such changes.