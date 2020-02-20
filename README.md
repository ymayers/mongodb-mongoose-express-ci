# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)  SOFTWARE ENGINEERING IMMERSIVE

## Getting Started

- Fork
- Clone

# Express API - Continuous Integration

What is Continuous Integration? Let's take 10min to read:

> Contemplate: What is CI? Why is it important? Should I used it?

- https://www.atlassian.com/continuous-delivery/continuous-integration
- https://www.thoughtworks.com/continuous-integration
- https://martinfowler.com/articles/continuousIntegration.html

We are going to hit the ground running with this lesson. Up until this point you have created an express json api full crud (even a bit of unit testing). So let's skip the basics and dive right in!

We will now create our directory structure (quickly).

Copy this entire code snippet and paste it into your terminal and hit return:

```sh
cd mongodb-mongoose-express-ci
npm init -y && 
npm install mongoose express body-parser morgan cors faker && 
npm install --save-dev nodemon jest supertest && 
echo "
/node_modules
.DS_Store
.env" >> .gitignore &&
mkdir db models seed routes controllers tests &&
touch server.js app.js db/index.js models/{user,project}.js seed/userProjects.js routes/index.js controllers/index.js tests/{base,routes}.test.js &&
code .
```

Let's create our database connection:

mongodb-mongoose-express-ci/db/index.js
```js
const mongoose = require('mongoose')

let MONGODB_URI = process.env.PROD_MONGODB || 'mongodb://127.0.0.1:27017/projectsDatabase'

mongoose
    .connect(MONGODB_URI, { useUnifiedTopology: true, useNewUrlParser: true })
    .then(() => {
        console.log('Successfully connected to MongoDB.')
    })
    .catch(e => {
        console.error('Connection error', e.message)
    })

const db = mongoose.connection

module.exports = db
```

Next let's create our user model:

mongodb-mongoose-express-ci/models/user.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const User = new Schema(
    {
        first_name: { type: String, required: true },
        last_name: { type: String, required: true },
        email: { type: String, required: true }
    },
    { timestamps: true },
)

module.exports = mongoose.model('users', User)
```

And now we can create our project model:

mongodb-mongoose-express-ci/models/project.js
```js
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const Project = new Schema(
    {
        title: { type: String, required: true },
        image_url: { type: String, required: true },
        description: { type: String, required: true },
        github_url: { type: String, required: true },
        deployed_url: { type: String, required: true },
        user_id: { type: Schema.Types.ObjectId, ref: 'user_id' }
    },
    { timestamps: true },
)

module.exports = mongoose.model('projects', Project)
```

Awesome, let's populate our database with some projects that belong to users:

mongodb-mongoose-express-ci/seed/userProjects.js
```js
const db = require('../db')
const Project = require('../models/project')
const User = require('../models/user')
const faker = require('faker')

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const main = async () => {
    const users = [...Array(25)].map(user => (
        {
            first_name: faker.name.firstName(),
            last_name: faker.name.lastName(),
            email: faker.internet.email()
        }
    ))
    const createdUsers = await User.insertMany(users)
    console.log('Created users!')
    
    const projects = [...Array(100)].map(item => {
        const user = createdUsers[Math.floor(Math.random() * 25)]
        return {
            title: faker.lorem.sentence(),
            image_url: faker.internet.url(),
            description: faker.lorem.paragraph(),
            github_url: faker.internet.url(),
            deployed_url: faker.internet.url(),
            user_id: user._id
        }
    })
    await Project.insertMany(projects)
    console.log('Created projects!')
}



const run = async () => {
    await main()
    db.close()
}

run()
```

Execute the seed file:

```sh
node seed/userProjects.js
```

Check in MongoDB Compass that the database was created with the correct data.

C00L. Data is good. Let's move on to create our express app.

mongodb-mongoose-express-ci/app.js
```js
const express = require('express');
const cors = require('cors')
const bodyParser = require('body-parser');
const logger = require('morgan');
const routes = require('./routes');

const app = express();

app.use(cors())
app.use(bodyParser.json())
app.use(logger('dev'))

app.use('/api', routes);

module.exports = app
```

And the code to instantiate our express app:

mongodb-mongoose-express-ci/server.js
```js
const app = require('./app.js')
const db = require('./db')

const PORT = process.env.PORT || 3000

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

app.listen(PORT, () => console.log(`Listening on port: ${PORT}`))
```

This a good time to edit our scripts:

```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
    "dev": "nodemon server.js",
    "start": "node server.js"
  },
```

Let's define our express routes:

mongodb-mongoose-express-ci/routes/index.js
```js
const { Router } = require('express')
const controllers = require('../controllers')
const router = Router()

router.get('/', (req, res) => res.send('This is root!'))

router.post('/users', controllers.createUser)
router.get('/users', controllers.getAllUsers)
router.put('/users/:id', controllers.updateUser)
router.delete('/users/:id', controllers.deleteUser)

router.get('/users/:user_id/projects/:item_id', controllers.getProjectByUserId)
router.get('/users/:user_id/projects', controllers.getProjectsFromUser)
router.get('/users/:id/projects', controllers.getProjectsFromUser)
router.post('/users/:user_id/projects', controllers.createProject)
router.get('/projects/:id', controllers.getProject)
router.put('/projects/:id', controllers.updateProject)
router.delete('/projects/:id', controllers.deleteProject)

module.exports = router
```

> Notice we are creating nested routes! A user can have project(s).

Let's now create our controllers based on our nested routes.

mongodb-mongoose-express-ci/controllers/index.js
```js
const db = require('../db')
const User = require('../models/user')
const Project = require('../models/project')

db.on('error', console.error.bind(console, 'MongoDB connection error:'))

const createUser = async (req, res) => {
    try {
        const user = await new User(req.body)
        await user.save()
        return res.status(201).json(user)
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}

const getAllUsers = async (req, res) => {
    try {
        const users = await User.find()
        return res.status(200).json(users)
    } catch (error) {
        return res.status(500).send(error.message)
    }
}

const updateUser = async (req, res) => {
    try {
        const { id } = req.params;
        await User.findByIdAndUpdate(id, req.body, { new: true }, (err, user) => {
            if (err) {
                res.status(500).send(err);
            }
            if (!user) {
                res.status(500).send('User not found!');
            }
            return res.status(200).json(user);
        })
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

const deleteUser = async (req, res) => {
    try {
        const { id } = req.params;
        const deleted = await User.findByIdAndDelete(id)
        if (deleted) {
            return res.status(200).send("User deleted");
        }
        throw new Error("User not found");
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

const getProjectsFromUser = async (req, res) => {
    try {
        const { user_id } = req.params
        const projects = await Project.find({ user_id: user_id })
        if (projects) {
            return res.status(200).json(projects)
        }
        return res.status(404).send('User with the specified ID does not exist')
    } catch (error) {
        return res.status(500).send(error.message)
    }
}

const getProjectByUserId = async (req, res) => {
    try {
        const { user_id, project_id } = req.params
        const project = await Project.findOne({ user_id: user_id, _id: project_id })
        if (project) {
            return res.status(200).json(project)
        }
        return res.status(404).send('Project with the specified ID does not exist')
    } catch (error) {
        return res.status(500).send(error.message)
    }
}

const createProject = async (req, res) => {
    try {
        const user = await User.findById(req.params.user_id)
        const project = await new Project(req.body)
        project.user_id = user._id
        await project.save()
        return res.status(201).json(project)
    } catch (error) {
        return res.status(500).json({ error: error.message })
    }
}
const getProject = async (req, res) => {
    try {
        const project = await Project.findById(req.params.id)
        res.send(project)
    } catch (error) {
        return res.status(500).send(error.message)
    }
}

const updateProject = async (req, res) => {
    try {
        const { id } = req.params;
        await Project.findByIdAndUpdate(id, req.body, { new: true }, (err, project) => {
            if (err) {
                res.status(500).send(err);
            }
            if (!project) {
                res.status(500).send('Project not found!');
            }
            return res.status(200).json(project)
        })
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

const deleteProject = async (req, res) => {
    try {
        const { id } = req.params;
        const deleted = await Project.findByIdAndDelete(id)
        if (deleted) {
            return res.status(200).send("Project deleted");
        }
        throw new Error("Project not found");
    } catch (error) {
        return res.status(500).send(error.message);
    }
}

module.exports = {
    createUser,
    getAllUsers,
    getProject,
    updateUser,
    deleteUser,
    getProjectsFromUser,
    getProjectByUserId,
    createProject,
    updateProject,
    deleteProject
}
```

> Notice above how we implement the nested logic in our controllers

Ok. Enough code. Let's run our server and test our endpoints.

```sh
npm run dev
```

Test all endpoints in your browser and using Postman.

Good? Let's make sure it stays that way by writing some unit tests for our endpoints.

Create your base test:

mongodb-mongoose-express-ci/tests/base.test.js
```js
describe('Initial Test', () => {
    it('should test that 1 + 1 === 2', () => {
        expect(1 + 1).toBe(2)
    })
})
```

And finally our routes tests:

mongodb-mongoose-express-ci/tests/routes.test.js
```js
const request = require('supertest')
const app = require('../app.js')
const mongoose = require('mongoose')
const databaseName = 'projectsTestDatabase'
const Project = require('../models/project')
const User = require('../models/user')
const faker = require('faker')

beforeAll(async () => {
    const MONGODB_URI = `mongodb://127.0.0.1/${databaseName}`
    await mongoose.connect(MONGODB_URI, { useUnifiedTopology: true, useNewUrlParser: true })

    const users = [...Array(25)].map(user => (
        {
            first_name: faker.name.firstName(),
            last_name: faker.name.lastName(),
            email: faker.internet.email()
        }
    ))
    const createdUsers = await User.insertMany(users)
    console.log('Created users!')
    
    const projects = [...Array(100)].map(item => {
        const user = createdUsers[Math.floor(Math.random() * 25)]
        return {
            title: faker.lorem.sentence(),
            image_url: faker.internet.url(),
            description: faker.lorem.paragraph(),
            github_url: faker.internet.url(),
            deployed_url: faker.internet.url(),
            user_id: user._id
        }
    })
    await Project.insertMany(projects)
    console.log('Created projects!')
})

let project, user

describe('Projects API', () => {
    it('should show all users', async done => {
        const res = await request(app).get('/api/users')
        expect(res.statusCode).toEqual(200)
        user = res.body[0]
        expect(res.body[0]).toHaveProperty('_id')
        done()
    }),
    it('should show all projects', async done => {
        const res = await request(app).get(`/api/users/${user._id}/projects`)
        expect(res.statusCode).toEqual(200)
        expect(res.body[0]).toHaveProperty('_id')
        done()
    }),
    it('should create a new project', async done => {
        const res = await request(app)
            .post(`/api/users/${user._id}/projects`)
            .send({
                title: 'Test Project',
                image_url: 'http://www.testing.com',
                description: 'http://www.testing.com',
                github_url: 'http://www.testing.com',
                deployed_url: 'http://www.testing.com',
                user_id: user._id
            })
        expect(res.statusCode).toEqual(201)
        expect(res.body).toHaveProperty('_id')
        project = res.body._id
        done()
    }),
    it('should show a project', async done => {
        const res = await request(app).get(`/api/projects/${project}`)
        expect(res.statusCode).toEqual(200)
        expect(res.body).toHaveProperty('_id')
        done()
    }),
    it('should update a project', async done => {
        const res = await request(app)
            .put(`/api/projects/${project}`)
            .send({
                title: 'Update Test Project',
                image_url: 'http://www.testing.com',
                description: 'http://www.testing.com',
                github_url: 'http://www.testing.com',
                deployed_url: 'http://www.testing.com',
                user_id: user._id
            })
        expect(res.statusCode).toEqual(200)
        expect(res.body).toHaveProperty('_id')
        done()
    }),
    it('should delete a project', async done => {
        const res = await request(app).del(`/api/projects/${project}`)
        expect(res.statusCode).toEqual(200)
        expect(res.text).toEqual("Project deleted")
        done()
    })
})

afterAll(async () => {
    await mongoose.connection.db.dropDatabase()
    await mongoose.connection.close()
})
```

Let's edit our script file:

mongodb-mongoose-express-ci/package.json
```js
  "scripts": {
    "test": "jest tests --detectOpenHandles",
    "dev": "nodemon server.js",
    "start": "node server.js"
  },
  "jest": {
    "testEnvironment": "node"
  },
```

Test it!

```sh
npm test
```

PASS!

## Continuous Integration

We will now setup Continuous Integration (CI). The idea is that anytime we push changes to GitHub, it will kickoff a build of our project on Travis CI with the latest changes. Travis CI will run our tests and either pass or fail the tests. Additionally, we will integrate [Coveralls](https://coveralls.io) to check test coverage on our codebase - the idea is that all new features we push up to GitHub should be paired with a Unit Test.

Ok enough words, let's start by integrating [Travis CI](https://travis-ci.org).

### Integrating Travis CI

![](travis-ci.png)

##

1. First you will need to signup at [Travis CI](https://travis-ci.org).

2. Make sure you have pushed all changes up to GitHub.

3. Once you've created an account with Travis CI, add your repo.

4. Activate your repo.

Now we need to setup the [Travis CI](https://travis-ci.org) config file:

```sh
touch .travis.yml
```

Add the following to .travis.yml:

```yml
language: node_js
node_js:
  - 'stable'
install: npm install
services:
  - postgresql
before_script:
  - npm run db:create:test
script: npm test
after_success: npm run coverage
```

> This tells [Travis CI](https://travis-ci.org) what to do. You can see in this config that we're telling [Travis CI](https://travis-ci.org) to runs tests, if the test succeed, then run [Coveralls](https://coveralls.io)

### Integrating Coveralls

![](coveralls.jpg)

##

Before we can run [Travis CI](https://travis-ci.org) we need to setup [Coveralls](https://coveralls.io) so let's do that:

```sh
touch .coveralls.yml
```

1. Go to the [Coveralls](https://coveralls.io) website and sign up.

2. Add your repo. 

3. Click on your repo inside the coveralls website. Copy the repo_token. Paste it inside of .coveralls.yml

4. npm install coveralls

5. Scroll to the bottom of the coveralls website on your repo page, copy the markdown for the coveralls badge. Paste on line 1 of your readme.

Change your package.json:

```js
  "scripts": {
    "test": "cross-env NODE_ENV=test jest  --testTimeout=10000 --detectOpenHandles --forceExit",
    "pretest": "cross-env NODE_ENV=test npm run db:reset",
    "db:create:test": "cross-env NODE_ENV=test npx sequelize-cli db:create",
    "db:drop:test": "cross-env NODE_ENV=test npx sequelize-cli db:drop",
    "start": "node server.js",
    "dev": "nodemon server.js",
    "db:reset": "npx sequelize-cli db:drop && npx sequelize-cli db:create && npx sequelize-cli db:migrate && npx sequelize-cli db:seed:all",
    "coverage": "npm run db:drop:test && npm run db:create:test && npm run pretest && jest --coverage && cat ./coverage/lcov.info | coveralls"
  },
```

Finally, you can now push changes up and it will kick off a Travis CI build.

Check for a successful build.

![](travisci-pass.png)

##

You will also see your Coveralls badge in your README.md updated.

![](coveralls-status.png)

##

### Adding Deployment to the Travis CI Build

So we get all the tests to pass. Then what? Well that means the app is ready for production - its ready to deploy. Let's have Travis CI kickoff a Heroku deployment if all tests pass.

> Make sure you're on the `master` branch!

1. `heroku create your-heroku-app-name`
2. `heroku buildpacks:set heroku/nodejs`
3. `heroku addons:create heroku-postgresql:hobby-dev --app=your-heroku-app-name`

Now let's configure Travis CI to do the deployment upon our test successfully passing.

First, we have to install the [Travis CI CLI](https://github.com/travis-ci/travis.rb#readme) tool:

```sh
gem install travis -v 1.8.10
```

We are going to use the Travis CLI to setup our Travis Heroku config:

```sh
travis setup heroku
```

> Make sure you encrypt your heroku api key! More info on running heroku commands on travis [here](https://docs.travis-ci.com/user/deployment/heroku/#running-commands)


Ok, our .travis.yml file should now look something like this:

```yml
language: node_js
node_js:
- stable
install: npm install
services:
- postgresql
before_script:
- npm run db:create:test
script: npm test
after_success: npm run coverage
deploy:
  provider: heroku
  api_key:
    secure: your-heroku-encrypted-api-key
  app: your-heroku-app-name
  on:
    repo: your-github-usernam/your-repo
  run:
    - "npx sequelize-cli db:migrate"
    - "npx sequelize-cli db:seed:all"
```

```sh
git status
git commit -am "add heroku deployment to travis build"
git push
```

And watch the Travis build for:

1. Successful tests.
1. Successful Heroku deployment.
1. Successful Sequelize migrations on Heroku.
1. Successful Sequelize seeding on Heroku.

Once the build is complete, test the endpoints on Heroku to confirm: 

- https://your-heroku-app-name.herokuapp.com/api/users
- https://your-heroku-app-name.herokuapp.com/api/users/1

**Excellent!**

> ✊ **Fist to Five**

## Feedback

> [Take a minute to give us feedback on this lesson so we can improve it!](https://forms.gle/vgUoXbzxPWf4oPCX6)
