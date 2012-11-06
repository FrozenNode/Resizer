# Changes to original Resizer

## Goal of the fork
The fork originated from another Laravel bundle I made called Multup
The goal of my fork is to allow for multiple images to be uploaded, validated, and then resized. It keeps the original method of using resizer intact (I believe).
The fork also allows for the user to define a callback function after each image is uploaded. 

## Using the new methods

1. Instead of passing the $file resource into the ::open method, it will be passed into the upload method
		
	 Resizer::open()
		->upload( string 'file', string 'image|max:3000|mimes:jpg,gif,png', string 'imgs/originals/', opt bool)

Param 1
 'file' is the name of the input HTML element. Same name you would use in Input::file( 'file' ) 
Param 2
 A string of validation functions... same as the laravel validation class
Param 3
 A string containing the path relative to /public/ to save your uploaded files. Currently it does not support not saving them, but perhaps that will be added
Param 4
 Boolean on whether or not to randomize the filename. I havent tested setting this to false :-p It will set it to a random 32 character alphanum if true
 I plan on adding a method to define your own randomization function

2. The after_upload method. This should be called before the upload method. It can be any callable function. 
 The function will get passed an array containing the following variables
  path: string containing path relative to /public/ to access the image
  filename: string containing the full filename (including extension) of the uploaded image
  resized: this will always be an empty string
  
  To call the method
	Resizer::open()
		->after_upload(function($args){ return 'lolcats'; })
		->upload('file', 'image|max:3000|mimes:jpg,gif,png', 'imgs/originals/')

	or
	Resizer::open()
		->after_upload('Model::method')
		->upload('file', 'image|max:3000|mimes:jpg,gif,png', 'imgs/originals/'')

3. Calling the resize function remains unchanged

4. Save return and parameters are slightly changed
Instead of the first parameter in the save method call being the path & the filename to save the resized image as, it will just be the path with the trailing slash

	->save( 'images/thumbs/' , 90 );

Currently, the resized image will be given the same name as the original upload, so put the resized in a different folder.

If you are using the upload method, the save will now return an array of arrays containing the same variables listed in step 2
  path:  string containing path relative to /public/ to access the image
  filename: string containing the full filename (including extension) of the uploaded image
  resized: this will be true or false

If you have specified an after_upload callback there will also be a key in each array with the name callback_result
So your return would look something like...

	[{"path":"imgs\/originals\/muffins.png", "filename":"muffins.png","resized":true,"callback_result":true}]

If you are not using the upload method, the save function will return true/false as usual.
		
## To do

Better error reporting and checking
I will be adding in a way to add multiple resizes to be made for each image. I am thinking the sytax will be something like 

	Instead of 
	 ->resize( 200 , 200 , 'crop' )
	 Pass in an array to the first argument
	 ->resize( array( array( 200, 200, 'crop' ), array( 300, 100, 'auto') ) )