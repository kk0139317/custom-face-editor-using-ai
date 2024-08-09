# General components

## 1. Face Detector
### 1.1 RetinaFace 
This face detector is used in the default workflow. It's built directly into the stable-diffusion-webui, so no additional software installation is required, reducing the chance of operational issues.

This component is implemented using [facexlib/detection](https://github.com/xinntao/facexlib/blob/master/facexlib/detection/__init__.py).

#### Name
- RetinaFace

#### Implementation
- [RetinafaceDetector](retinaface_detector.py)

#### Recognized UI settings
- Advanced Options - (1) Face Detection - Face detection confidence

#### Configuration Parameters (in JSON)
- N/A

#### Returns
- tag: "face"
- attributes: N/A
- landmarks: 5 (both eyes, the nose, and both ends of the mouth)

#### Usage in Workflows
- [default.json](../../workflows/default.json)


---

### 1.2 lbpcascade_animeface
This face detector is designed specifically for anime/manga faces.

This component is implemented using [lbpcascade_animeface](https://github.com/nagadomi/lbpcascade_animeface).

#### Name
- lbpcascade_animeface

#### Implementation
- [LbpcascadeAnimefaceDetector](lbpcascade_animeface_detector.py)

#### Recognized UI settings
- N/A

#### Configuration Parameters (in JSON)
- `min_neighbors` (integer, default: 5): Specifies the minimum number of neighboring rectangles that make up an object face. A higher number results in fewer detections but higher quality, while a lower number improves detection rate but may result in more false positives.

#### Returns
- tag: "face"
- attributes: N/A
- landmarks: N/A

#### Usage in Workflows
- [lbpcascade_animeface.json](../../workflows/examples/lbpcascade_animeface.json)

---

## 2. Face Processor

### 2.1 img2img
This is the implementation used for enhancing enlarged face images in the default workflow.

#### Name
- img2img

#### Implementation
- [Img2ImgFaceProcessor](img2img_face_processor.py)

#### Recognized UI settings
- Prompt for face
- Advanced Options - (3) Recreate the Faces - Denoising strength for face images

#### Configuration Parameters (in JSON)
- `pp` (string): positive prompt.
- `np` (string): negative prompt.

#### Usage in Workflows
- [default.json](../../workflows/default.json)

---

### 2.2 Blur
This face processor applies a Gaussian blur to the detected face region. The intensity of the blur can be specified using the `radius` parameter in the 'params' of the JSON configuration. The larger the radius, the more intense the blur effect.

![Blur](../../images/inferencers/blur.jpg)

#### Name
- Blur

#### Implementation
- [BlurProcessor](blur_processor.py)

#### Recognized UI settings
- N/A

#### Configuration Parameters (in JSON)
- `radius` (integer, default: 20): The radius of the Gaussian blur filter.

#### Usage in Workflows
- [blur.json](../../workflows/examples/blur.json)
- [blur_non_center_faces.json](../../workflows/examples/blur_non_center_faces.json)
- [blur_young_people.json](../../workflows/examples/blur_young_people.json)

---

### 2.3 NoOp
This face processor does not apply any processing to the detected faces. It can be used when no face enhancement or modification is desired, and only detection or other aspects of the workflow are needed.

#### Name
- NoOp

#### Implementation
- [NoOpProcessor](no_op_processor.py)

#### Recognized UI settings
- N/A

#### Configuration Parameters (in JSON)
- N/A


---

## 3. Mask Generator

### 3.1 BiSeNet
This operates as the Mask Generator in the default workflow. Similar to RetinaFace Face Detector, it's integrated into the stable-diffusion-webui, making it easy to use without the need for additional software installations. It generates a mask using a deep learning model (BiSeNet) based on the face area of the image. Unique to BiSeNet, it can recognize the "Affected areas" specified by the user, including 'Face', 'Hair', 'Hat', and 'Neck'. This option will be invalid with other mask generators. It includes a fallback mechanism where if the BiSeNet fails to generate an appropriate mask, it can fall back to the `VignetteMaskGenerator`. 

This component is implemented using [facexlib/parsing](https://github.com/xinntao/facexlib/blob/master/facexlib/parsing/__init__.py).

#### Affected areas - Face
![Face](../../images/inferencers/bisenet-face.jpg)

#### Affected areas - Face, Hair
![Face, Hair](../../images/inferencers/bisenet-face-hair.jpg)

#### Affected areas - Face, Hair, Hat
![Face, Hair, Hat](../../images/inferencers/bisenet-face-hair-hat.jpg)

#### Affected areas - Face, Hair, Hat, Neck
![Face, Hair, Hat, Neck](../../images/inferencers/bisenet-face-hair-hat-neck.jpg)


#### Name
- BiSeNet

#### Implementation
- [BiSeNetMaskGenerator](bisenet_mask_generator.py)

#### Recognized UI settings
- Use minimal area (for close faces)
- Affected areas
- Mask size

#### Configuration Parameters (in JSON)
- `fallback_ratio` (float, default: 0.25): Extent to which a mask must cover the face before switching to the fallback generator. Any mask covering less than this ratio of the face will be replaced by a mask generated by the **Vignette** MaskGenerator`.

#### Usage in Workflows
- [default.json](../../workflows/default.json)

---

### 3.2 Vignette
This mask generator creates a mask by applying a Gaussian (circular fade-out effect) to the face area. It is less computationally demanding than deep-learning-based mask generators and can consistently produce a mask under conditions where deep-learning-based mask generators such as BiSeNet or YOLO may struggle, such as with unusual face orientations or expressions. It serves as the fallback mask generator for the BiSeNet Mask Generator when it fails to generate an appropriate mask. 

![Vignette](../../images/inferencers/vignette-default.jpg)

#### Name
- Vignette

#### Implementation
- [VignetteMaskGenerator](vignette_mask_generator.py)

#### Recognized UI settings
- Use minimal area (for close faces)

#### Configuration Parameters (in JSON)
- `sigma` (float, default: -1): The spread of the Gaussian effect. If not specified or set to -1, a default value of 120 will be used when `use_minimal_area` is set to True, and a default value of 180 will be used otherwise.
- `keep_safe_area` (boolean, default: False): If set to True, a safe area within the ellipse around the face will be preserved entirely within the mask, preventing the fade-out effect from being applied to this area.

---

### 3.3 Ellipse
This option draws an ellipse around the detected face region to generate a mask.

![Ellipse](../../images/inferencers/ellipse.jpg)

#### Name
- Ellipse

#### Implementation
- [EllipseMaskGenerator](ellipse_mask_generator.py)

#### Recognized UI settings
- Use minimal area (for close faces)

#### Configuration Parameters (in JSON)
- N/A

---

### 3.4 Rect
This is a simplistic implementation that uses the detected face region as a direct mask.

![Rect](../../images/inferencers/face-area-rect.jpg)

#### Name
- Rect

#### Implementation
- [FaceAreaMaskGenerator](face_area_mask_generator.py)

#### Recognized UI settings
- N/A

#### Configuration Parameters (in JSON)
- N/A

---

### 3.5 NoMask
This option generates a "mask" that is simply an all-white image of the same size as the input face image. It essentially does not mask any part of the image and can be used in scenarios where no masking is desired.

![NoMask](../../images/inferencers/no-mask.jpg)

#### Implementation
- [NoMaskGenerator](no_mask_generator.py)

#### Name
- NoMask

#### Recognized UI settings
- N/A

#### Configuration Parameters (in JSON)
- N/A

#### Usage in Workflows
- [blur_non_center_faces.json](../../workflows/examples/blur_non_center_faces.json)
- [blur_young_people.json](../../workflows/examples/blur_young_people.json)
