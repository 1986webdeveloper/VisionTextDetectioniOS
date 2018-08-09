# VisionTextDetectioniOS

TextDetection

Among many of the powerful frameworks Apple released at this year’s WWDC, the Vision framework was one of them. you can have your app perform a number of powerful tasks such as identifying faces and faical features (ex: smile, frown, left eyebrow, etc.), barcode detection, classifying scenes in images, object detection and tracking, and horizon detection.

Features 

	⁃	Scanning of any documents.
	⁃	no dependency.
	⁃	scene any text 

Requirements

Swift 4.0
IOS 10.0+
Xcode 9.x

Installation

To integrate TextDetection into your Xcode project no need any pods.

Usage

1)	import this two framework in your viewcontroller

	import Vision
	import AVFoundation

2)	Make sure that your view controller conforms to the AVCaptureVideoDataOutputSampleBufferDelegate protocol

class YourViewController: UIViewController, AVCaptureVideoDataOutputSampleBufferDelegate {
}

3)	Implement of delegate function

func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, from connection: AVCaptureConnection) {
guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else {return}
        
        var requestOptions:[VNImageOption : Any] = [:]
        
        if let camData = CMGetAttachment(sampleBuffer, kCMSampleBufferAttachmentKey_CameraIntrinsicMatrix, nil) {
            requestOptions = [.cameraIntrinsics:camData]
        }
        
        let imageRequestHandler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, orientation: CGImagePropertyOrientation(rawValue: 6)!, options: requestOptions)
        
        do {
            try imageRequestHandler.perform(self.requests)
        } catch {
            print(error)
        }
}

4)	add Vision setup function 

 // MARK: - Vision Setup
    func setupVision() {
        let textRequest = VNDetectTextRectanglesRequest(completionHandler: self.textDetectionHandler)
        textRequest.reportCharacterBoxes = true
        
        self.requests = [textRequest]
    }
    
    func textDetectionHandler(request: VNRequest, error: Error?) {
        guard let observations = request.results else {print("no result"); return}
        
        let result = observations.map({$0 as? VNTextObservation})
        
        DispatchQueue.main.async() {
            self.previewView.layer.sublayers?.removeSubrange(1...)
            for region in result {
                guard let rg = region else {continue}
                self.drawRegionBox(box: rg)
                if let boxes = region?.characterBoxes {
                    for characterBox in boxes {
                        self.drawTextBox(box: characterBox)
                    }
                }
            }
        }
    }

5) add camera setup function

func setupCamera() {
        previewView.session = session
        let availableCameraDevices = AVCaptureDevice.DiscoverySession(deviceTypes: [.builtInWideAngleCamera], mediaType: AVMediaType.video, position: .back)
        
        var activeDevice: AVCaptureDevice?
        
        for device in availableCameraDevices.devices as [AVCaptureDevice]{
            if device.position == .back {
                activeDevice = device
                break
            }
        }
        
        do {
            let camInput = try AVCaptureDeviceInput(device: activeDevice!)
            
            if session.canAddInput(camInput) {
                session.addInput(camInput)
            }
        } catch {
            print("no camera")
        }
        session.sessionPreset = .high
        guard auth() else {return}
        
        let videoOutput = AVCaptureVideoDataOutput()
        
        videoOutput.setSampleBufferDelegate(self, queue: DispatchQueue(label: "buffer queue", qos: .userInteractive, attributes: .concurrent, autoreleaseFrequency: .inherit, target: nil))
        
        if session.canAddOutput(videoOutput) {
            session.addOutput(videoOutput)
        }
        previewView.videoPreviewLayer.videoGravity = .resize
        session.startRunning()
    }

![alt text](http://dev.acquaintsoft.com/textdetection.gif)
