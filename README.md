# ExpressJS Notes

## Step-1 Initialize the project
create a new folder
```bash
npm init -y
```
create a index.js and .env file

## Step-2 Install required packages
```bash
npm install express sequelize mysql2 multer jsonwebtoken bcrypt cors
```
express: makes backend very easy
mysql2: used for querying MYSql db
sequelize: ORM to connect with db and work with DB
multer: incoming file management middleware or for accepting multipart form data
jsonwebtoken: to send a encrypted token to front end for secure connection
bcrypt: hashing passwords
cors: cross origin resourse sharing: allowing different methods and urls for backend to interact, otther wise frontend will not connect

also change package.json 
```json
"scripts": {
  "dev": "node --env-file=.env --watch index.js"
}
```
## step-3 Create project structure
```
backend/
|-config/
  |-db.js
|-models/
  |-Model.js
|-controllers/
  |-modelController.js
|-routes/
  |-modelRoutes.js
|-index
|-.env
```

## Step-4 inside config/db.js
 from https://sequelize.org/docs/v6/getting-started/  copy connecting to database code with option 3 only , your db.js should look like this
```js
//config/db.js
 const { Sequelize } = require('sequelize');
 const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: /* one of 'mysql' | 'postgres' | 'sqlite' | 'mariadb' | 'mssql' | 'db2' | 'snowflake' | 'oracle' */
});
module.exports=sequelize
```
you may test the connection by copying and creating a function in index.js, make sure to import sequilize from config/db.js
```js
//index.js
const authenticate=async()=>{
try {
  await sequelize.authenticate();
  console.log('Connection has been established successfully.');
} catch (error) {
  console.error('Unable to connect to the database:', error);
}}

//call
authenticate()
```

## step-5 inside models/ create a model
using singular captial first letter Naming , create a model file, ex: User.js
from  https://sequelize.org/docs/v6/core-concepts/model-basics/ copy extending model part into your User.js file
donot copy the second line , its for createing a new db connection, but we have already defined the connection in db.js
```js
//config/User.js
const { Sequelize, DataTypes, Model } = require('sequelize');

class User extends Model {}

User.init(
  {
    // Model attributes are defined here
    firstName: {
      type: DataTypes.STRING,
      allowNull: false,
    },
    lastName: {
      type: DataTypes.STRING,
      // allowNull defaults to true
    },
  },
  {
    // Other model options go here
    sequelize, // We need to pass the connection instance
    modelName: 'User', // We need to choose the model name
  },
);

module.exports=User
```

## Step-5 Synchronise with MYSql to create the table inside our database
this better should be done in index.js file after importing all models

```js
//index.js
//make sure to import model for the syncronisation to work properly
//const User=require('./models/User.js') 

const syncDb=async()=>{
await sequelize.sync({ force: true });
console.log('All models were synchronized successfully.');
}
//call function
syncDB()// make sure to comment this out once you have your tables else on every save it will drop all tables and create //all from start
```

## step-6 with middlewares inplace your index.js should look
```js
//index.js
const express = require('express')
const sequelize=require('./config/db.js') 
const app = express()
const port = process.env.PORT
const userRouter=require('./routes/UserRouter')
const User=require('./models/User.js')

//middlewares for accepting data
app.use(express.json())
app.use(app.use(express.urlencoded({ extended: true }))

//

const authenticate=async()=>{
    try {
        await sequelize.authenticate();
        console.log('Connection has been established successfully.');
      } catch (error) {
        console.error('Unable to connect to the database:', error);
      }
}
// authenticate()

//routes
app.use('/users',userRouter)

async function syncDb(){

  await sequelize.sync({ force: true });
  console.log('All models were synchronized successfully.');
}
// syncDb()


app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})

```
## Step-7 Create Controller
this is where the actual coding, or logic will come, getting data from db, updating, deleting and creating data.
```js
//controller/userContoller.js
const User=require('../models/User')
const getAllUsers= async (req,res)=>{
    const users=await User.findAll()
    res.json(users)
  }
//create other functions
//export all functions
  module.exports={getAllUsers}
```

## Step -8 Create Routes
create userRouter.js in routes folder
for routes to work you need this basic syntax
```js
// routes/userRouter.js
const express = require('express')
const router = express.Router()
const {getAllUsers} =require('../controllers/UserController') //import all created functions
//all routes come here
//ex:
router.get('/', getAllUsers)

//create other routes and assing them related functions from controller 

module.exports = router

```

## run this and check with api client
```bash
npm run dev
```

if you get erros solve the erros and  you have a proper backend
