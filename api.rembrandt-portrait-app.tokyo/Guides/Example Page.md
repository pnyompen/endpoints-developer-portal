# Python API Tutorial: Getting Started with APIs

## Install packages


```python
!pip install requests pillow
```


```python
import requests
from PIL import Image
from matplotlib import pyplot as plt
import base64
import io
from urllib.parse import urljoin
import pickle
import os
```

## open input image


```python
INPUT_IMAGE_PATH = 'test_img.png'
```


```python
im = Image.open(INPUT_IMAGE_PATH)
plt.imshow(im)
```




    <matplotlib.image.AxesImage at 0x118995730>




![png](output_6_1.png)


## resize the input image within 1024x1024px.  


```python
MAX_UPLOAD_SIZE = (1024, 1024)
im.thumbnail(MAX_UPLOAD_SIZE, Image.LANCZOS)
plt.imshow(im)
```




    <matplotlib.image.AxesImage at 0x117fd10a0>




![png](output_8_1.png)


## comvert the input image to base64


```python
buffer = io.BytesIO()
im.save(buffer, 'jpeg', quarity=90)
img_binary = buffer.getvalue()
img_base64 = base64.b64encode(img_binary).decode('utf-8')
```

## crop face and get facial metadata


```python
API_BASE_URL = 'https://api.rembrandt-portrait-app.tokyo'
API_KEY = os.environ.get('API_KEY')
```


```python
url = urljoin(API_BASE_URL, '/v1/detect')
payload = {'image': img_base64}
params = {'key': API_KEY}
res = requests.post(url, json=payload, params=params)
res
```




    <Response [200]>




```python
res.json()['meta']
```




    {'status': 'ok', 'processing_time': '3.516625'}



This cropped image can be used as a preview to improve the user experience.


```python
cropped_image = res.json()['data']['image']
cropped_image = Image.open(io.BytesIO(base64.b64decode(cropped_image)))
plt.imshow(cropped_image)
```




    <matplotlib.image.AxesImage at 0x118aeb460>




![png](output_16_1.png)


The metadata will be used in next step.  
Please save it.  

If the input image does not change, the metadata will not change.


```python
facial_metadata = res.json()['data']['metadata']
```

**If the input image does not change, the metadata will not change and you can skip the above steps**

## convert image to rembrandt style


```python
STYLE_ID = 'rembrandt_portrait-of-a-woman-1632_0'
SKIN_TONE = '0'
```


```python
url = urljoin(API_BASE_URL, '/v1/convert')
payload = {
    'style': STYLE_ID,
    'metadata': facial_metadata,
    'skin_tone': SKIN_TONE
}
params = {'key': API_KEY}
res = requests.post(url, json=payload, params=params)
res
```




    <Response [200]>




```python
res.json()['meta']
```




    {'status': 'ok', 'processing_time': '3.188128'}




```python
converted_image = res.json()['data']['image']
converted_image = Image.open(io.BytesIO(base64.b64decode(converted_image)))
plt.imshow(converted_image)
```




    <matplotlib.image.AxesImage at 0x118b2b100>




![png](output_24_1.png)

