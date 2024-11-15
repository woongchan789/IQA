# Image Quality Assessment

Last modified: 2023.02.03

WHAT IS IQA?
---
이미지의 품질을 정량화하려는 영역입니다. 이미지들이 뿌옇거나 살짝 회전이 되있다던지  
그러한 이미지들을 인간의 시각 시스템(Human Visual System, 이하 HVS)에 의한 평가에서 벗어나  
통계적 특성이나 알고리즘에 의해 정량화하려는 노력입니다.  

대표적으로 PSNR, SSIM, BRISQUE가 존재하는데 최근엔 CNN이나 딥러닝 모델을  
활용한 IQA 모델들이 많이 등장하고 있습니다.

BREFOLA
---
BREFOLA는 Blind/Referenceless model via Fourier transform and Laplacian filter의 약자로  
제가 제안하는 IQA 모델입니다.  
크게 Fourier transform과 Laplacian filter를 적용한 NR(No-Reference)방식입니다([FR, NR에 대한 설명](https://bskyvision.com/entry/IQA-CNN-%EA%B8%B0%EB%B0%98-%EC%9D%B4%EB%AF%B8%EC%A7%80%ED%92%88%EC%A7%88%ED%8F%89%EA%B0%80-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EC%A0%95%EB%A6%AC)).  
제가 제안한 BREFOLA는 기존 IQA의 영역에서의 다양한 왜곡의 형태(압축, blur, white noise, ...)에  
적용이 가능한 모델은 아니며 blur 왜곡에만 적용이 가능하고 주행환경에서 적합한 즉, 적용도가 낮은 모델입니다.  
본 연구가 자율주행자동차의 카메라 센서 신뢰성 향상을 목적으로 연구가 진행이 되었기에  
blur만을 왜곡현상으로 보았고 주행환경에서 발생한 문제점을 해결하고자  
Laplacian filter을 사용하였기에 아직까지도 많은 부분이 부족한 지표입니다.  

<p align="center"><img src="https://user-images.githubusercontent.com/75806377/216756264-706b7d18-4f03-47f6-a424-06d1fa59cf2d.png" height="500px" width="800px"></p> 

Fourier Transform
---
제가 제안한 BREFOLA는 먼저 이미지 픽셀 밝기값의 변화를  
파형으로 보고 Fourier transform을 통해 주파수 영역대로 변환합니다.  
이 행위 자체가 의미하는 바는 픽셀의 변화가 적은 배경부분과  
픽셀의 변화가 큰 edge 부분을 각각 저주파와 고주파 성분으로 보겠다는 의미이며  
실제로 이미지의 blur의 강도를 높일수록 저주파 성분이 많아지고 고주파 성분이 줄어듦을 확인하였습니다.  
Fourier Transform을 통해 저주파 성분이 많아지고 고주파 성분이 줄어듦을 확인하였기에 이를 정량화하고자  
Fourier Transform > Log Transform > Shifted spectrum 을 통해 분포를 2D Image로 나타내었습니다.  
Shifted spectrum 이미지 내에 threshold를 설정해 그 이상인 픽셀 값들의 개수를 count함으로써 정량화하였습니다.

Problems
---
Fourier Transform을 통해 blur 강도를 높일수록 정량화한 value가  
작아지는 경향을 우선 catch할 수 있었으나 문제점이 존재했습니다.  
밤에 고속도로를 타보신 경험이 있을까요? 하늘은 깜깜하고.. 상향등만 도로를 비추는..  
그런 이미지의 경우 카메라 센서가 고장난 것이 아니라  
단지 밤이라는 어두운 조도에 의한 이미지이나 푸리에 변환을 하게되면  
깜깜한 하늘이 대부분을 차지하는 이미지의 경우 픽셀 변화가 거의 없어 저주파 성분이 엄청 많아지게 됩니다.  
또한 주행환경에 있어 표지판, 신호등 등 주행하는데 있어 몇 가지의 중요한 요소는 존재하나  
나무나 건물을 장식하는 조명 즉, 일루미네이션 등이나 창문이 엄청많은 건물 등이 이미지에 존재하게 되면  
고주파 성분이 너무 많아지게 되어 정상 카메라 센서로 찍힌 이미지들에도 불구하고 차이가 너무 커지게 됩니다.

- 낮 시간대, 조명 장식이 많은 즉, 이미지 복잡도가 높은 이미지  

<p align="center"><img src="https://user-images.githubusercontent.com/75806377/216619152-bb5c397b-5d22-4921-a5eb-103ea7b505c9.png" height="300px" width="500px"></p>

- 밤 시간대, 요소가 거의 없는 즉, 이미지 복잡도가 낮은 이미지  

<p align="center"><img src="https://user-images.githubusercontent.com/75806377/216619156-0a14eb03-eff6-4ef0-b889-afb646885150.png" height="300px" width="500px"></p>

Laplacian filter
---
위의 문제점을 해결하기위해 불필요한 고주파 성분을 어느정도 배제하고자  
Fourier Transform에서 정량화하였던 value에 고주파 성분의 양을 나눠줌으로써 해결하고자 하였습니다.  
고주파 성분을 효과적으로 정량화하기위해 HPF(High-Pass Filter)와 convolution을 진행했습니다.  
Laplacian filter, Canny filter, Prewitt filter 등 다양한 HPF와 convolution을 하여 실험을 해본결과  
고주파 성분을 잘 정량화할 수 있는 filter는 Laplacian filter였기에 원본 이미지와 Laplacian filter를  
convolution한 edge 그림을 활용하여 정량화하고자 하였습니다.  
edge 그림 내의 픽셀값들을 모두 더하여 정량화한 후 Fourier Transform에서 정의한 value와 단위가 맞지 않기에  
scaling작업으로 edge 그림 내의 픽셀값들을 모두 더한 값에 제곱근을 씌워 BREFOLA로 정의하였습니다.  

<p align="center"><img src="https://user-images.githubusercontent.com/75806377/216619159-65b6c607-0bf6-48e0-87cf-39dec37b667f.png" height="300px" width="500px"></p>

Results
---
그래프는 Average filter를 N x N 으로 강도를 높여가면서 적용한 전후 그래프입니다.  
X축이 1X1인 경우가 원본 이미지의 경우인데 BREFOLA를 적용하기전 값들의 분산이 큰 것을 알 수 있는데  
BREFOLA를 적용한 후 확실히 줄어든 것을 확인할 수 있습니다.  
또한 HVS를 기준으로 blurry하다고 평가되는 4X4 일 때를 보면  
4X4 최대값이 1X1 즉, 원본 이미지의 최소값보다 작으므로  
정상 원본 이미지를 blur가 적용된 이미지들과 구별이 가능함을 의미합니다.  
한 마디로 고장난 카메라로 찍은 낮시간대에 건물 많고 복잡한 이미지와  
정상 카메라로 찍은 깜깜한 고속도로 이미지를 구별할 수 있음을 의미합니다!

<p align="center"><img src="https://user-images.githubusercontent.com/75806377/216756267-ce75ec69-3265-49d5-844a-2eaaab46a55c.png" height="500px" width="800px"></p>

Limitations
---
IQA영역에서 한 가지 왜곡(blurry)에만 국한된 적용도가 낮은 모델이며 주행환경이라는  
특수한 상황을 바탕으로 연구가 진행되었기에 주행이미지에만 검증작업을 진행하였으며  
피사체가 강아지, 사람인 이미지의 경우에는 다소 부적합합니다.  
IQA영역의 다양한 왜곡에도 적용이 가능하며 다양한 이미지에 적용할 수 있도록  
발전시켜 HVS를 따라갈 수 있도록..! 노력하겠습니다.

