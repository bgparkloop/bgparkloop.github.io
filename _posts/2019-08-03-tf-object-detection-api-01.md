---
layout: post
title: "[Study] Object Detection: Tensorflow API 사용법"
categories: [deep learning]
tags: [object detection, tensorflow]
fullview: true
comments: true
---

### 1. Tensorflow Object Detection API Installation
#### 1.1. Tensorflow Models 다운로드
git을 사용해서 Tensorflow model API를 받는다.
=> git clone https://github.com/tensorflow/models

#### 1.2. protobuffer 다운로드
- Windows
아래 링크에서 필요로 하는 버전을 다운받는다.
ex) protoc-3.9.1-win32.zip 다운 받으면 압축파일 안에 zip파일이 있음
https://github.com/protocolbuffers/protobuf/releases/tag/v3.9.1

- Linux
간단하게 아래 명령어로 설치
pip install protobuf

#### 1.3. 필요 Library 설치
- pillow
- lxml
- jupyter
- matplotlib
- 기타 등등..

#### 1.4. models setup
git으로 받은 API폴더가 있는 곳으로 가면 models폴더가 있다. 그 안에 research 폴더로 이동 후,

- Windows
YOUR_PROTOBUF_PATH/protoc.exe object_detection/protos/*.proto --python_out=.

- Linux
protoc object_detection/protos/*.proto --python_out=.

명령어 실행 후, protos 폴더에 pb2.py 파일들이 생성되었으면, 다음 명령어 실행

- python setup.py build
- python setup.py install

#### 1.5. 환경변수 설정
아래와 같이 models 폴더가 있는곳(YOUR_API_PATH)을 기준으로 2개의 경로를 환경변수로 설정
1. YOUR_API_PATH/models/research
2. YOUR_API_PATH/models/research/slim

- Windows
환경변수 설정으로 가서 PATH에 추가한다.

- Linux
~/.bashrc 파일에 export 환경변수를 추가하고, source ~/.bashrc로 적용.
또는 /etc/profile에 같은 방식으로 진행하면 모든 계정에 적용됨.

#### 1.6. 동작확인
그대로 models/research 폴더에서
python object_detection/builders/model_builder_test.py를 실행하여 아래와 같은 문장이 나오면 정상 동작.

- (Ran 22 tests in 0.102s)

---
### 2. 학습 준비
#### 2.1. Pretrained model 다운로드
아래 주소로 들어가 학습하고자 하는 모델을 다운받는다.
- https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md

예시)
**'ssd_mobilenet_v2_coco'**를 받고 압축을 풀면 **'ssd_mobilenet_v2_coco_2018_03_29'**와 같은 폴더에 다음과 같은 것들이 있다.
다른 것들은 크게 신경안써도 되고 pipline.config만 따로 옮겨주면 된다.

#### 2.2. 데이터 / 설정 준비
데이터를 넣어둘 폴더와 학습결과를 저장할 폴더를 생성한다.
아래는 예시이다.
- 데이터 폴더 : images
  - images폴더 안에는 train과 test 폴더를 만들어 나중에 필요한 데이터를 넣는다.
- 학습결과 폴더 : training
  - training 폴더에는 앞서 언급한 pipline.config파일을 옮겨두고, ssdlite_mobilenet_v2_coco.config와 같이 원하는 이름으로 변경해준다.(꼭 변경안해도 상관은 없다) 그 후 아래 부분들을 수정한다.

~~~
# 검출하고자하는 클래스 수로 수정한다.
num_classes: 90
...
# 기본 사이즈를 그대로 쓰거나 원한다면 사이즈를 변경한다.
image_resizer {
      fixed_shape_resizer {
        height: 300
        width: 300
      }
    }
...
# batch_size는 메모리가 되는만큼 크게 잡는 것이 학습효율에 좋다.
train_config {
  batch_size: 24
# data_augmentation_options의 경우는 뒤에 따로 설명한다.
...
# fine_tune_checkpoint에는 우리가 받았던 pretrained model이 있는 폴더 경로를 PATH_TO_BE_CONFIGURED에 넣으면 된다.
fine_tune_checkpoint: "PATH_TO_BE_CONFIGURED/model.ckpt"
# num_steps는 학습을 몇 번 반복할지 결정한다. 기본이 20만번인데, class수와 문제 난이도에 따라 크기는 달리하면 된다.
num_steps: 200000
...

#
train_input_reader {
  label_map_path: "PATH_TO_BE_CONFIGURED/mscoco_label_map.pbtxt"
  tf_record_input_reader {
    input_path: "PATH_TO_BE_CONFIGURED/mscoco_train.record"
  }
}
eval_config {
  num_examples: 8000
  max_evals: 10
  use_moving_averages: false
}
eval_input_reader {
  label_map_path: "PATH_TO_BE_CONFIGURED/mscoco_label_map.pbtxt"
  shuffle: false
  num_readers: 1
  tf_record_input_reader {
    input_path: "PATH_TO_BE_CONFIGURED/mscoco_val.record"
  }
}
~~~

 - training 폴더 안에는 학습하고자 하는 label정보도 들어가야 한다. 그러므로 labelmap.pbtxt라는 문서를 하나 만들어 클래스 수만큼 다음과 같이 넣는다.

~~~
# Class 수만큼 item을 정의해야함. id 0번은 배경이므로 1번부터 시작.
item {
    id: 1
    name: 'normal'
}
item {
    id: 2
    name: 'abnormal'
}
~~~



