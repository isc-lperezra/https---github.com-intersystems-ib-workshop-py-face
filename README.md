# workshop-py-face
Example about an IRIS production using Embedded Python to recognize and identify faces in photos

You can find more in-depth information in https://learning.intersystems.com.

# What do you need to install? 
* [Git](https://git-scm.com/downloads) 
* [Docker](https://www.docker.com/products/docker-desktop) (if you are using Windows, make sure you set your Docker installation to use "Linux containers").
* [Docker Compose](https://docs.docker.com/compose/install/)
* [Visual Studio Code](https://code.visualstudio.com/download) + [InterSystems ObjectScript VSCode Extension](https://marketplace.visualstudio.com/items?itemName=daimor.vscode-objectscript)

# Setup
Build the image we will use during the workshop:

```console
$ git clone https://github.com/intersystems-ib/workshop-py-face
$ cd workshop-py-face
$ docker-compose build
```

The current project is self-deployable and it doesn't require further configuration. The Python libraries required are installed during the deployment of the container.

# Example

The main purpose of this example is to ingest images from a specific folder, identify the faces in the images and check any possible match of the faces in a group of images used as repository.

In order to accomplish this goal we are going to use the functionality available in IRIS to use a language like Python in our ObjectScript classes.

We are going to use two models pre-trained to simplify the code, the first one, `mobilenet_graph.pb` will help us to recognize all the faces in our image. The second one, `facenet_keras_weights.h5` will allow us to compare the face found with the faces in the images of our repository.

## Nomenclator Production 
* Run the containers we will use in the workshop:
```
docker-compose up
```
Automatically an IRIS instance will be deployed and a production will be configured and run to read JPG files from /shared/JPG folder.

* Open the [Management Portal](http://localhost:52773/csp/sys/UtilHome.csp).
* Login using the default `superuser`/ `SYS` account.
* Click on [Nomenclator Production](http://localhost:52773/csp/nomenclator/EnsPortal.ProductionConfig.zen?$NAMESPACE=NOMENCLATOR&$NAMESPACE=NOMENCLATOR&) to access the sample production that we are going to use. You can access also from NOMENCLATOR namespace through *Interoperability > User > Configure > Production*.
* Click on Nomenclator.FaceChecker business service and review the configuration.
  * You will notice that is using EnsLib.File.InboundAdapter as the adapter to read all the files in /shared/JPG folder.
  * Also you will see five new settings:
    * KnownsPath: the folder that will be used as a repository of knowns faces by the Python code. The Python method will compare the found image with the images in this folder. 
    * UnknownsPath: any file found in /shared/JPG will be copied to this folder to be treated by the Python method.
    * ResultsPath: in this folder we are going to create a copy of the original image with a mark around any face found out. In case of match with any other face the mark will be of green color, otherwise the color will be red. 
    * ModelDetectorFilePath: location in the container of `mobilenet_graph.pb` file.
    * ModelRecognitionFilePath: location in the container of `facenet_keras_weights.h5` file.
* Try to insert a new record into Patient table (you can do it from Adminer), you will see a new message received in the production.

Opening Visual Studio Code you will be able to manage the different folders configured, you will be able to copy jpg images from your own computer into the container the business operation used to read the object created for each row in Patient's table and the object definition

## Python Method
If you are interested in know more about the Python functionality you can take a look to [this article](https://www.codificandobits.com/blog/tutorial-reconocimiento-facial-python/) in spanish that I used to build the entire functionality.

## How to make it run
You only need to copy the file `travi.jpg` located in `/shared/test` folder into `/shared/JPG` folder. I've included 4 photos of John Travolta as the face known, so any new photo added into `/shared/JPG` of John Travolta should return in the Events Log a message with the match and you will see in `/shared/results` folder your photo with the face enmarked in green.

In the case that you put into `/shared/JPG` the other file `nicolas.jpg` you will see an empty line in the Events Log and the JPG created into `/shared/results` will contain the face enmarked in red.