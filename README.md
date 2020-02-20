# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)  SOFTWARE ENGINEERING IMMERSIVE

## Getting started

1. Fork
1. Clone

# Express, Mongoose, & MongoDB

Let's start!

```sh
cd express-mongo-using-router
npm init -y && npm install mongoose
mkdir db models seed
touch db/index.js models/plant.js seed/plants.js
```

Now let's open up Visual Studio Code and write some code:

```sh
code .
```

Inside our `db` folder we are going to use Mongoose to establish a connection to our MongoDB `plantsDatabase`:

express-mongo-using-router/db/index.js
```js
const mongoose = require('mongoose')

mongoose
    .connect('mongodb://127.0.0.1:27017/plantsDatabase', { useUnifiedTopology: true, useNewUrlParser: true })
    .then(() => {
        console.log('Successfully connected to MongoDB.');
      })
    .catch(e => {
        console.error('Connection error', e.message)
    })

const db = mongoose.connection

module.exports = db
```

Although, MongoDB is schema-less, Mongoose allows us to write a schema for our plant model which makes it nice to know what is a plant in our database and what a plant "looks" like in our database:

express-mongo-using-router/models/plant.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const Plant = new Schema(
    {
        name: { type: String, required: true },
        description: { type: String, required: true },
        image: { type: String, required: true },
    },
    { timestamps: true },
)

module.exports = mongoose.model('plants', Plant)
```

Cool. We have a "blueprint" for what a plant is. Let's now use it and create plants.

express-mongo-using-router/seed/plants.js
```js
const db = require('../db')
const Plant = require('../models/plant')

// Connect to the database
db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const plants = [
    new Plant({ name: 'Aloe Vera', description: 'Aloe vera is a succulent plant species of the genus Aloe. An evergreen perennial, it originates from the Arabian Peninsula, but grows wild in tropical, semi-tropical, and arid climates around the world. It is cultivated for agricultural and medicinal uses.', image: 'https://upload.wikimedia.org/wikipedia/commons/4/4b/Aloe_vera_flower_inset.png' }),
    new Plant({ name: 'Snake Plant', description: 'Sansevieria trifasciata is a species of flowering plant in the family Asparagaceae, native to tropical West Africa from Nigeria east to the Congo. It is most commonly known as the snake plant, Saint George's sword, mother-in-law's tongue, and viper's bowstring hemp, among other names.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Snake_Plant_%28Sansevieria_trifasciata_%27Laurentii%27%29.jpg/2560px-Snake_Plant_%28Sansevieria_trifasciata_%27Laurentii%27%29.jpg' }),
    new Plant({ name: 'Areca palm', description: 'Dypsis lutescens, also known as golden cane palm, areca palm, yellow palm, or butterfly palm, is a species of flowering plant in the family Arecaceae, native to Madagascar and naturalized in the Andaman...', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/4/4b/Dypsis_lutescens1.jpg/1280px-Dypsis_lutescens1.jpg' }),
    new Plant({ name: 'Spider Plant', description: 'Chlorophytum comosum, often called spider plant but also known as airplane plant, St. Bernard's lily, spider ivy, ribbon plant, and hen and chickens is a species of perennial flowering plant. It is native to tropical and southern Africa, but has become naturalized in other parts of the world, including western Australia.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/b1/Hierbabuena_0611_Revised.jpg/1920px-Hierbabuena_0611_Revised.jpg' }),
    new Plant({ name: 'Dracaena', description: 'Dracaena is a genus of about 120 species of trees and succulent shrubs. In the APG IV classification system, it is placed in the family Asparagaceae, subfamily Nolinoideae (formerly the family Ruscaceae). It has also formerly been separated (sometimes with Cordyline) into the family Dracaenaceae or placed in the Agavaceae (now Agavoideae).', image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Dracaena_draco.jpg' }),
    new Plant({ name: 'Weeping Fig', description: 'Ficus benjamina, commonly known as weeping fig, benjamin fig or ficus tree, and often sold in stores as just ficus, is a species of flowering plant in the family Moraceae, native to Asia and Australia. It is the official tree of Bangkok.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/9/9d/Ficus_benjamina2.jpg/1280px-Ficus_benjamina2.jpg' }),
    new Plant({ name: 'Peace Lily', description: 'Spathiphyllum is a genus of about 40 species of monocotyledonous flowering plants in the family Araceae, native to tropical regions of the Americas and southeastern Asia. Certain species of Spathiphyllum are commonly known as spath or peace lilies.', image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/b/bd/Spathiphyllum_cochlearispathum_RTBG.jpg/1024px-Spathiphyllum_cochlearispathum_RTBG.jpg' })
]
const newPlants = () => {
    plants.forEach(async plant => await plant.save())
}

const run = async () => {
    await newPlants()
    console.log("Created plants!")
}

run()
```

Awesome, so this plants "seed" file above is a script that, once executed, connects to the Mongo database and creates 7 plants in the plants collection.

Let's execute our plants seed file:

```sh
node seed/plants.js
```

So how do we know if it worked? We could drop into the `mongo` interactive shell and check:

```sh
mongo
> use plantsDatabase
> db.plants.find()
```

Create a .gitignore file `touch .gitignore`!

```sh
/node_modules
.DS_Store
```

Cool, enough Mongoose. Now, Express. Let's install Express and Nodemon for development:

```sh
npm install express
npm install nodemon -D
```
And now let's setup our express folders:

```sh
mkdir routes controllers
touch server.js routes/index.js controllers/index.js
```

Modify your package.json file:

```js
....
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon server.js"
  },
....
```

Let's setup the root route:

./routes/index.js
```js
const { Router } = require('express');
const router = Router();

router.get('/', (req, res) => res.send('This is root!'))

module.exports = router;
```

Inside of server.js:
```js
const express = require('express');
const routes = require('./routes');

const PORT = process.env.PORT || 3000;

const app = express();

app.use('/api', routes);

app.listen(PORT, () => console.log(`Listening on port: ${PORT}`))
```

Test the route:
```sh
npm start
```

Test the root endpoint in your browser: http://localhost:3000/api/

Good, now let's work on the controller. The controller is where we will set up all of our logic e.g. what does the API do when we want to create a new plant? Update a plant? etc.

./controllers/index.js
```js
const Plant = require('../models/plant');

const createPlant = async (req, res) => {
    try {
        const plant = await Plant.create(req.body);
        return res.status(201).json({
            plant,
        });
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}

module.exports = {
    createPlant,
}
```

Remember we will need the express body-parser middleware to access the req.body object so:

```sh
npm i body-parser
```

Add the following lines of code to the top of server.js:

```js
const bodyParser = require('body-parser');
app.use(bodyParser.json())
```

Cool. We have the logic to create a new plant. Now let's create a route on our server to connect the request with the controller:

./routes/index.js:
```js
const { Router } = require('express');
const controllers = require('../controllers')
const router = Router();

router.get('/', (req, res) => res.send('This is root!'))

router.post('/plants', controllers.createPlant)

module.exports = router;
```

Make sure your json api server is running:
```sh
npm start
```

Use Postman (POST) method to test the create route (http://localhost:3000/api/plants):

```js
{
    "name": "Test Plant",
    "description": "Test Description",
    "image": "https://testimage.com/plant.png"
}
```

Awesome! Now I want to create a controller method to grab all the plants from the database:

./controllers/index.js
```js
const Plant = require('../models/plant');

const createPlant = async (req, res) => {
    try {
        const plant = await Plant.create(req.body);
        return res.status(201).json({
            plant,
        });
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}

const getAllPlants = async (req, res) => {
    try {
        const plants = await Plant.findAll({
            include: [
                {
                    model: Project
                }
            ]
        });
        return res.status(200).json({ users });
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

module.exports = {
    createPlant,
    getAllPlants
}
```

Add the following route to your ./routes/index.js file:
```js
router.get('/plants', controllers.getAllPlants)
```

Open http://localhost:3000/api/plants in your browser or do a GET request in Postman.

Nice, now let's add the ability to find a specific plant:

./controllers/index.js
```js
const getUserById = async (req, res) => {
    try {
        const { id } = req.params;
        const user = await User.findOne({
            where: { id: id },
            include: [
                {
                    model: Project
                }
            ]
        });
        if (user) {
            return res.status(200).json({ user });
        }
        return res.status(404).send('User with the specified ID does not exists');
    } catch (error) {
        return res.status(500).send(error.message);
    }
}
```

Add it to the export:

./controllers/index.js
```js
module.exports = {
    createUser,
    getAllUsers,
    getUserById
}
```

Add the route:

./routes/index.js
```js
router.get('/users/:id', controllers.getUserById)
```

Test it! http://localhost:3000/api/plants/2

This is a good point to integrate better logging. Right now, if we check our terminal when we hit the http://localhost:3000/api/plants/2 endpoint we see the raw SQL that was executed. For debugging purposes and overall better logging we're going to use an express middleware called morgan:

```sh
npm install morgan
```

Add the following to your server.js file:
```js
const logger = require('morgan');
app.use(logger('dev'))
```

Let's see the result:
```sh
npm start
open http://localhost:3000/api/plants/2
```

You should now see in your terminal something like this:
```sh
GET /api/plants/2 304 104.273 ms
```

That's morgan!

So we can now create plants, show all plants, and show a specific plant. How about updating a plant and deleting a plant?

./controllers/index.js
```js
const updateUser = async (req, res) => {
    try {
        const { id } = req.params;
        const [updated] = await User.update(req.body, {
            where: { id: id }
        });
        if (updated) {
            const updatedUser = await User.findOne({ where: { id: id } });
            return res.status(200).json({ user: updatedUser });
        }
        throw new Error('User not found');
    } catch (error) {
        return res.status(500).send(error.message);
    }
};

const deleteUser = async (req, res) => {
    try {
        const { id } = req.params;
        const deleted = await User.destroy({
            where: { id: id }
        });
        if (deleted) {
            return res.status(204).send("User deleted");
        }
        throw new Error("User not found");
    } catch (error) {
        return res.status(500).send(error.message);
    }
};
```

Make sure your exports are updated:
```js
module.exports = {
    createUser,
    getAllUsers,
    getUserById,
    updateUser,
    deleteUser
}
```

Let's add our routes:

./routes/index.js
```js
router.put('/users/:id', controllers.updateUser)
router.delete('/users/:id', controllers.deleteUser)
```

Test update (PUT) in Postman. Your request body in Postman will have to look something like this:

http://localhost:3000/api/plants/3

```js
{
    "name": "Update Plant Test",
    "description": "Test Description",
    "image": "https://testimage.com/plant.png"
}
```

Test delete (DEL) in Postman using a URL like this http://localhost:3000/api/users/3

Success! We built a full CRUD JSON API in Express, Mongoose, and Mongo using Express Router!


### Deployment

Let's deploy our app to [heroku](https://devcenter.heroku.com/articles/heroku-cli#download-and-install).

First we need to update our package.json:

```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js",
    "dev": "nodemon server.js",
    "db:reset": "npx sequelize-cli db:drop && npx sequelize-cli db:create && npx sequelize-cli db:migrate && npx sequelize-cli db:seed:all"
  },
```

> Make sure you're on the `master` branch!

1. `heroku create your-heroku-app-name`
2. `heroku buildpacks:set heroku/nodejs`
3. `heroku addons:create heroku-postgresql:hobby-dev --app=your-heroku-app-name`
4. `git status`
5. `git commit -am "add any pending changes"`
6. `git push heroku master`
7. `heroku run npx sequelize-cli db:migrate`
8. `heroku run npx sequelize-cli db:seed:all`

> Having issues? Debug with the Heroku command `heroku logs --tail` to see what's happening on the Heroku server.

Test the endpoints :)

> https://your-heroku-app-name.herokuapp.com/api/projects

> https://your-heroku-app-name.herokuapp.com/api/projects/1

**Excellent!**

> ✊ **Fist to Five**

## Feedback

> [Take a minute to give us feedback on this lesson so we can improve it!](https://forms.gle/vgUoXbzxPWf4oPCX6)













































Add the code:

```js
const express = require('express');
const PORT = process.env.PORT || 3000;

const app = express();

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

app.listen(PORT, () => {
  console.log(`Express server listening on port ${PORT}`);
});

app.get('/', (req, res) => {
  res.send("This is root!");
});
```

Let's make sure our server works:

```sh
node server.js
open localhost:3000
```

Awesome! Next we want to be able to access our Plant model from within the models folder.
Add the following to the top of your server.js file:

```js
const Plant = require('./models/plant');
```

Let's create the route to show all plants:

```js
app.get('/plants', async (req, res) => {
    const plants = await Plant.find()
    res.json(plants)
})
```

Restart the server and test the route:

```sh
node server.js
```

Try it in your browser: http://localhost:3000/plants

Now I would like to see a specific plant.
Let's say you type http://localhost:3000/plants/5e385d110909f66c6404fbc9 then our API should respond with the product where id equals 2. Express let's us do this via the `req.params` object:

```js
app.get('/plants/:id', async (req, res) => {
  const { id } = req.params
  const plant = await Plant.findById(id)
  res.json(plant)
})
```

What if the plant does not exist in the database? We would get an ugly error message. We can handle this by using a try/catch block:

```js
app.get('/plants/:id', async (req, res) => {
    try {
        const { id } = req.params
        const plant = await Plant.findByPk(id)
        if (!plant) throw Error('Plant not found')
        res.json(plant)
    } catch (e) {
        console.log(e)
        res.send('Plant not found!')
    }
})
```

Does it work? Restart the server and test the route.

```sh
node server.js
```

Open http://localhost:3000/plants/5e385d110909f66c6404fbc9 in your browser.

Success!

![](http://www.winsold.com/sites/all/modules/winsold/images/checkmark.svg)
