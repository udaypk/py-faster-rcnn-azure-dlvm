# py-faster-rcnn-azure-dlvm
Faster RCNN to deploy on Microsoft Azure DLVM. Original R-CNN code forked from https://github.com/rbgirshick/py-faster-rcnn.  
You can check out https://medium.com/@udayp/caffe-faster-r-cnn-on-microsoft-azure-cloud-part2-567ecdff6be0 for instructions on installing the DLVM and configuring it. I'm reproducing parts of it below.

### Configure Faster R-CNN on Azure DLVM GPU instance
0. Spawn an Azure DLVM following instructions at https://medium.com/@udayp/caffe-faster-r-cnn-on-microsoft-azure-cloud-part1-c273621e24ba
1.	After logging into the virtual machine, you first need to switch to root user. If you have logged in using ssh, type the below command in the terminal. If you logged in using graphical interface X2Go, first open a terminal window and type the below command in the terminal window.

	```sudo -s```

	***Useful tip: If you use X2Go, after you copy text from your local machine, use Shift+Insert to paste in the terminal of cloud VM.***

2.	As mentioned in my previous article, DLVM comes pre-installed with some essential libraries like Python and we don’t need to install them. They are set up under anaconda environment. We need to switch to that environment using the command given below. 

	```source /anaconda/bin/activate root```

	***Step 1 and 2 are essential for anything you do on an Azure DLVM. If you want to install new packages or change any environment parameters, you need to run steps 1 and 2. Whenever you open a new terminal window, you need to run steps 1 and 2.***

3.	We need to install a few more packages to compile and run Caffe with OpenCV. Run the followings commands in the terminal. 

	```
	conda install -c auto easydict
	conda install -c anaconda readline
	conda install -c anaconda opencv 
	```

4.	It’s time to download and compile the code! In the terminal window, go to your desired directory and clone Caffe Faster R-CNN from Github using the command below. 

	```git clone --recursive https://github.com/udaypk/py-faster-rcnn-azure-dlvm.git```

	This code is forked from https://github.com/rbgirshick/py-faster-rcnn. To build this on DLVM, I made changes to the Makefile.config which are described at the end of this article.

5.	Step 4 creates a folder called py-faster-rcnn-azure-dlvm. First build Cython by executing the command given below (make command should be executed in py-faster-rcnn-azure-dlvm/lib folder). This step is optional as the repository already is built. But this may be necessary in the future if there are any changes in DLVM OS or hardware. 

	```
	cd py-faster-rcnn-azure-dlvm/lib
	make
	``` 

6.	Next step is to build caffe and pycaffe. Run the commands given below. 
	```
	cd ../caffe-fast-rcnn
	make -j6 && make pycaffe
	```
	First command is to navigate to py-faster-rcnn-azure-dlvm/caffe-fast-rcnn folder. Second command is for building caffe and pycaffe. You can build faster by using -j option for multi-threading. The option is -jn where n is the number of CPUs on your VM. For NC6 the option is -j6, for NC12 it is -j12 and so on.
7.	Now that Caffe R-CNN is built, you can test it using the pre-trained models shared by Ross Girshick. Run the following commands.
	```
	cd ..
	./data/scripts/fetch_faster_rcnn_models.sh
	./data/scripts/fetch_faster_rcnn_models.sh
	./tools/demo.py
	```
	First command is to go to py-faster-rcnn-azure-dlvm folder. Second one is for downloading pre-trained VGG16 ImageNet model. Run the command again to verify the checksum of the downloaded file. Next command is for running the demo script. After this, you will see images with boxes drawn around different objects.
8.	The most important step in ensuring the correct compilation of caffe on DLVM is setting the correct parameters in Makefile.config (located in py-faster-rcnn-azure-dlvm/caffe-fast-rcnn folder). Below are the most important changes.
  	1. As we use opencv, we need to un-comment the line OPENCV_VERSION := 3
  	2. Azure DLVM comes with pre-installed mkl library (Math Kernel Library). So, we need to change BLAS to mkl and include appropriate        directories. I faced a lot of difficulty trying to use atlas.
  	3. As Azure DLVM uses anaconda environment, we need to set PYTHON_INCLUDE to the anaconda environment.
  	4. Un-comment WITH_PYTHON_LAYER as we want to use python training and inference interfaces.
 
Following the above steps, I hope you can configure your Azure DLVM in a couple of hours. Do comment if you face any issue or if you have any other insights.
