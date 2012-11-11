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

3. Calling the resize function has changed a good deal.

Instead of having 3 parameters for width, height, and crop type, you will now pass in an array of arrays containing the info for each resize. Like so:
	
	$sizes = array( 
		array(200 , 200 , 'crop', 'public/images/thumbs/200/', 90 ), 
		array(300 , 300 , 'crop', 'public/images/thumbs/300/', 90 ), 
	);

The 4th parameter is the path to save the cropped image, and the 5th parameter is the image quality. So a complete call would now look something like
	
	$success = Resizer::open()
			->upload('file' ,  'image|max:3000|mimes:jpg,gif,png', 'tattoo_imgs/originals/' )
			->resize( $sizes );

4. The save function no longer exists as the saving is done in the resize function

Currently, the resized image will be given the same name as the original upload, so put the resized in a different folder.

The resize function will now return an array of arrays containing the same variables listed in step 2
  path:  string containing path relative to /public/ to access the image
  filename: string containing the full filename (including extension) of the uploaded image
  resized: an array of booleans

If you have specified an after_upload callback there will also be a key in each array with the name callback_result
So your return would look something like...

	[{"path":"imgs\/originals\/muffins.png", "filename":"muffins.png","resized":[true,true],"callback_result":true}]

If you are not using the upload method, the save function will return true/false as usual.
		
## To do

Better error reporting and checking
