# multer-image-uploader-tut
Tutorial using MERN stack to upload and retrieve images

Find the blog post [here](https://www.settletom.com/blog/uploading-images-to-mongodb-with-multer)

------------

Nowadays basic CRUD functionality is used in basically every application on the web. Most know how easy it is to store basic data types in databases, but with the more apps I have built the more I have seen how prevalent media types are in database storage and how most apps could put use to them. So let's look at how we can store a single image or multiple images in a database and then retrieve them to display.

## Technologies

- React
- Node
- Express
- Multer
- Mongoose
- MongoDB Atlas

We will build a basic React front end with a file picker as our interface to post to the backend. For the backend we will use Node with express to create an API that will interact with MongoDB's new database as a service Atlas, to store our images.

## Setting up MongoDB

First, if you have not already go ahead and make a MongoDB Atlas account or log in [here](https://www.mongodb.com/cloud/atlas)

Once logged in we will go ahead and create our cluster (Only one free cluster is allowed per account). For me when creating my cluster I stay will all the free options.

Now that the cluster has been created, choose the connect option in the cluster and add your IP address to the connections list and then create a database user. Next, choose to connect your application with Nodejs as the driver. Take note of this connection string as we will come back to it later.

![mongoconnect.jpg](https://media.graphcms.com/NRCGG2qaQw2zZD0tirwY)

## Front End App

Let's quickly get our front end up and running using [create-react-app](https://www.npmjs.com/package/create-react-app). If you have not already installed this globally in your cmd prompt run `npm i --g create-react-app` then navigate to the parent folder you want to store the project in and run `create-react-app image-uploader` once complete `cd image-uploader` and `yarn start` or `npm run start`

To make things easy we will the front end very quickly using bootstrap just to get an interface up and running. Go ahead and paste the two Bootstrap CDN links in the `public/index.html`

```js
   <link
      href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T"
      crossorigin="anonymous"
    />
   <script
      src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"
      integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM"
      crossorigin="anonymous"
    ></script>
```

Next, in `src/App.js` replace the return of `App()` with

```html
<div className="container">
  <div className="jumbotron">
    <h1 className="display-4">Image Uplaoder</h1>
    <p className="lead">
      This is a simple application to upload and retrieve images from a database
    </p>
    <hr className="my-4" />
  </div>
  <div className="input-group mb-3">
    <div className="custom-file">
      <input
        type="file"
        className="custom-file-input"
        id="inputGroupFile01"
        aria-describedby="inputGroupFileAddon01"
      />
      <label className="custom-file-label" htmlFor="inputGroupFile01">
        Choose file
      </label>
    </div>
  </div>
  <button type="button" className="btn btn-primary">
    Upload
  </button>
</div>
```

and create App into a React.Component

```js
class App extends React.Component {
  render() {
     return (...
```

Now on saving, with localhost:3000 open in a browser, our front end should look like this

![frontend.jpg](https://media.graphcms.com/MAfji8iLTg63vo4HyyI5)

## Back End

On the backend, we will get started by creating a folder called `server` in the root directory of our app in your terminal you can navigate to the root and run `mkdir server && cd server`. Next, in the folder, we will initialize with `npm init` accepting all the defaults. Now we can install all the necessary packages, go ahead and run `npm i body-parser express gridfs-stream mongoose multer multer-gridfs-storage && npm i -D nodemon`.

To get the server up and running `touch index.js` and in that file go ahead and add the following code.

```js
const express = require('express')
const bodyParser = require('body-parser')
const path = require('path')
const app = express()

app.use(bodyParser.json())

const port = 5000

app.listen(port, () => console.log(`Server started on port ${port}`))
```

This will get our server running on port 5000 with the command `nodemon index.js`

With the server up and running we can go ahead and start to add the app's functionality. First, let's go ahead and connect to our MongoDB using mongoose.

```js
//Connect to DB
const mongoose = require('mongoose')

const mongoURI = YOUR_MONGO_URI

const conn = mongoose.createConnection(mongoURI)

conn.once('open', () => {
  console.log('Connection Successful')
})
```

Before we start adding the routes we need to set up GridFS in order to properly save the media to mongo. In order to do this, we need to create the storage engine that will then be passed to multer.

First, we need to ensure some settings in our Atlas DB. Make sure the Database Access has a user with
readWriteAnyDatabase priveledges. Also in Network Access add an IP address of 0.0.0.0/0 to ensure you can upload from any IP.

![dbaccess.jpg](https://media.graphcms.com/6RrcdRmrQVq2z5euVlY0)

![ipaccess.jpg](https://media.graphcms.com/NN0Zp7JTPitkZ79rkTOi)

Now, to create the storage engine we will add this snippet in `index.js`

```js
// Create storage engine
const storage = new GridFsStorage({
  url: mongoURI,
  file: (req, file) => {
    return new Promise((resolve, reject) => {
      crypto.randomBytes(16, (err, buf) => {
        if (err) {
          return reject(err)
        }
        const filename = file.originalname
        const fileInfo = {
          filename: filename,
          bucketName: 'uploads',
        }
        resolve(fileInfo)
      })
    })
  },
})

const upload = multer({ storage })
```

The upload variable will not serve as out image object that will be sent to our DB.

Now we can go ahead and create a route for the image to post to. Below storage go ahead and add the route...

```js
app.post('/', upload.single('img'), (req, res, err) => {
  if (err) throw err
  res.status(201).send()
})
```

To test let's go ahead and go to our front end and set up the post function to send the image.

In `App.js` we need to add a function to append the file from our input to a new `formData` that will be sent over to our server to be saved in the DB.

create an `onClick` handler for the function Post...

```js
Post = e => {
  e.preventDefault()
  const file = document.getElementById('inputGroupFile01').files
  const formData = new FormData()

  formData.append('img', file[0])

  fetch('http://localhost:5000/', {
    method: 'POST',
    body: formData,
  }).then(r => {
    console.log(r)
  })
  console.log(file[0])
}
```

Now when we choose an image with the file picker and click upload this function will send to our server and get a response status of 201 which will be logged in the console along with our image uploaded!

When we check the Atlas DB in our collections we see in uploads.files our file is there!

![upload_result.jpg](https://media.graphcms.com/8Rb2OdNeRlGXNGAfTLYw)

## Show Image Uploaded

Lastly let's go ahead and show the image that we just uploaded on our front end, below our input form.

This will involve creating a function on the front end to retrieve it as well as a route on our server to `get`.

Let's start with the server route, pretty straight forward. We want to retrieve by filename. First, we need to create variables for the gridfs-stream package. Go ahead and append this to our db connection.

```js
let gfs

conn.once('open', () => {
  gfs = Grid(conn.db, mongoose.mongo)
  gfs.collection('uploads')
  console.log('Connection Successful')
})
```

This will allow us to use gfs to find the specific file we are looking for. Now we can go ahead and create the actual route.

```js
app.get('/:filename', (req, res) => {
  gfs.files.findOne({ filename: req.params.filename }, (err, file) => {
    // Check if file
    if (!file || file.length === 0) {
      return res.status(404).json({
        err: 'No file exists',
      })
    }

    // Check if image
    if (file.contentType === 'image/jpeg' || file.contentType === 'image/png') {
      // Read output to browser
      const readstream = gfs.createReadStream(file.filename)
      readstream.pipe(res)
    } else {
      res.status(404).json({
        err: 'Not an image',
      })
    }
  })
})
```

Next, we will work on the frontend to display this photo we just uploaded. This can be done with just a few lines of code in the Post function that we already created. But, first we need to create the `img` element that will hold the img. Under our upload button, I just added a simple `img` element with some inline styling.

```html
<img
          id="img"
          style={{
            display: "block"
          }}
        >
</img>
```

Now, all we do is add the following after our fetch in our existing Post function. This will set the `src` of the img to the route of the img in our DB.

```js
document
  .getElementById('img')
  .setAttribute('src', `http://localhost:5000/${file[0].name}`)
```

And BOOM! now when you upload an image it will immediately display on the front end and this is grabbing directly from our DB through the server.

![final.jpg](https://media.graphcms.com/1WbWU3lBSNCALqza2owY)

Thanks so much for following along! Let me know if there are any problems you run into or areas it can improve.

Here is the [repo](https://github.com/tsettle44/multer-image-uploader-tut) for reference.
