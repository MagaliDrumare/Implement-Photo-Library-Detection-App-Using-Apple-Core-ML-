 ![alt tag](https://github.com/MagaliDrumare/Implement-Photo-Library-Detection-App-Using-Apple-Core-ML-/blob/master/PhotoApp.gif)


# Implement Photo Library Detection App Using Apple Core ML
* Mastering Core ML for iOS on Udemy by Mohammad Azam : http://bit.ly/2ADSykK

```swift 


//  ViewController.swift

//  Created by DRUMARE on 14/11/2017.
//  Credits © Mastering Core ML for iOS Udemy 
//

import UIKit
import CoreML
import Vision

class ViewController: UIViewController,UINavigationControllerDelegate, 
UIImagePickerControllerDelegate {
    
    // create a variable imagePicker
    private var imagePicker = UIImagePickerController()
    
    // import the model GoogLeNetPLaces
    private var model = GoogLeNetPlaces()
    
    @IBOutlet weak var photoImageView: UIImageView!
    @IBOutlet weak var descriptionTextView: UITextView!
    

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
    
        // open the photoLibrary of the apple device
        self.imagePicker.sourceType = .photoLibrary
        self.imagePicker.delegate = self
    }
    // create the function to pick up the image
    func imagePickerController(_ picker: UIImagePickerController, 
    didFinishPickingMediaWithInfo info: [String : Any]) {
        //dismiss imagePickerController after the selection
        dismiss(animated: true, completion: nil)
    
        //keep the selected image as pickedImage
        guard let pickedImage = info[UIImagePickerControllerOriginalImage] as? UIImage else {
            return
        }
        // put the selected image in the photo container photoImageView
        self.photoImageView.image = pickedImage
        // apply to the pickedImage the processImage function that apply 
        //to each image the CoreML API with GoogLeNetPlaces model
        processImage(image:pickedImage)
    }
    
    // create the processImage function that take the image and will return the classification of the image into descriptionTextView
    private func processImage(image:UIImage) {
        
        // convert the image into CIImage format
        guard let ciImage = CIImage(image :image) else {
            fatalError("Unable to create a ciImage Object")
        }
    
        // create the visionModel to feed the API and obtain visionRequest
        // VNCoreMLModel is a container fore CoreML model used with visionRequest
        // put the model "private var model = GoogLeNetPlaces()" 
        //into the container : try? VNCoreMLModel(for:self.model.model)
        guard let visionModel = try? VNCoreMLModel(for:self.model.model) else {
            fatalError ("Unable to create a vision model")
        }
    
        // create the visionRequest that will return request or error
        // pass visionModel created to VNCoreMLRequest
        let visionRequest = VNCoreMLRequest(model: visionModel) {request, error in
            if error != nil{return
            }
            // VNClassificationObservation is the Scene classification 
            //information produced by an image analysis request.
            guard let results = request.results as? [VNClassificationObservation] else{
            return
            }
            
            // Instance properties of VNClassificationObservation : "var identifier: String"
            // Intance properties of VNObsevation var confidence: VNConfidence
            // VNClassificationObservation  inherits from VNObsevation
            // map the result of the request : results.map
            let classifications = results.map { observation in
                "\(observation.identifier) \(observation.confidence * 100)"
            }
            
            // put the result into the container descriptionTextView
            DispatchQueue.main.async {
            self.descriptionTextView.text = classifications.joined(separator: "\n")
            }
        
        }
        
        // create a visionRequestHandler which execute the request
        // initialize the visionRequestHandler
        // the visionRequestHandler take as parameter a ciImage 
        // (that's why we created "guard let ciImage")
        // orientation of the image is up
        // no another option [:]
        let visionRequestHandler = VNImageRequestHandler
        (ciImage: ciImage, orientation: .up, options: [:])
        
        // execute the visionRequest :  try! visionRequestHandler.perform([visionRequest])
        DispatchQueue.global(qos: .userInteractive).async {
            try! visionRequestHandler.perform([visionRequest])
        }
    
    
    }
    
    
    @IBAction func OpenLibraryButtonPressed(){
        // activate the imagePicker when the camera Button is pressed
         self.present(self.imagePicker, animated: true, completion: nil)
        
    }


}
```
