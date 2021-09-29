---
title: "[Study] Docker setup & simple example"
categories:
  - Docker
tags:
  - Docker
  - GPU
  - Flask
  - API
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

## Nvidia-docker2 설치 순서

- docker는 기본으로 설치되었다고 가정

```bash
# nvidia-docker repository 등록
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# package list update & install
sudo apt-get update
sudo apt-get install -y nvidia-docker2

# 간단 test
docker run --runtime=nvidia --rm nvidia/cuda:10.0-base nvidia-smi
```

## 새로운 컨테이너 만들기

- —shm-size : host와 컨테이너 사이의 shared memory 공간. 머신러닝 사용 시 충분한 메모리가 없으면 에러가 많이 발생함
- cuda:<version> : base, runtime, devel이 있는데, 필요한 상황에 따라 선택하면 됨. 일반적으로는 CUDA만 사용하면 되기 때문에 base면 충분

```bash
nvidia-docker run -d --shm-size <필요한 GB수> -it --name <CONTAINER_NAME> \
nvidia/cuda:<version>-<base, runtime, devel 중 선택> /bin/bash
```

## docker 기본 세팅

```bash
docker exec -it <CONTAINER_NAME> bash

# 기본 패키지 다운로드 세팅
sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
apt-get update
apt-get dist-upgrade -y

# 필수 패키지 설치
apt-get install -y wget vim git gcc build-essential

# anaconda install
# 필요에 따라 버전은 다르게
wget https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh
bash Anaconda3-2019.03-Linux-x86_64.sh
```

## Anaconda setting

```bash
# anaconda path 설정
export PATH=/root/anaconda3/bin:$PATH

conda create -n <env-name> python=3.6

# 그 외 설치하고 싶은대로!
```

```bash
nvidia-docker run -d --shm-size 2g -it --name {docker_name} nvidia/cuda:10.2-runtime \
 /bin/bash

docker exec -it {docker_name} bash

# 기본 패키지 다운로드 세팅
sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list
apt-get update
apt-get dist-upgrade -y

# 필수 패키지 설치
apt-get install -y wget vim git gcc build-essential

# anaconda install
# 필요에 따라 버전은 다르게
wget https://repo.anaconda.com/archive/Anaconda3-2019.03-Linux-x86_64.sh
bash Anaconda3-2019.03-Linux-x86_64.sh

# anaconda path 설정
export PATH=/root/anaconda3/bin:$PATH

conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch

# 컨테이너 외부에서 내부 python 파일 실행하기
docker exec {Container} sh -c "/root/anaconda3/bin/python3 {실행파일}"
```

### Dockerfile을 이용한 create

```bash
# Dockerfile이라는 파일이름으로 아래와 같이 만들기
# Base Image
FROM nvidia/cuda:10.2-cudnn7-runtime-ubuntu16.04
# 제작자 정보. 빌드랑 상관 없음
MAINTAINER bogyupark@doai.ai

# 세팅 할 
RUN sed -i 's/archive.ubuntu.com/ftp.daumkakao.com/g' /etc/apt/sources.list

RUN apt-get update && apt-get dist-upgrade -y
RUN apt-get install -y wget vim git gcc build-essential software-properties-common
RUN add-apt-repository -y ppa:fkrull/deadsnakes && apt-get update
RUN apt-get install -y python3.6 python3.6-dev libsm6 libxext6 libxrender-dev

RUN export PATH=$PATH:/usr/bin/python3.6
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2 && update-alternatives --install /usr/bin/python python /usr/bin/python3.6 1
RUN update-alternatives  --set python /usr/bin/python3.6 && update-alternatives --config python
RUN apt-get install -y python3-pip
RUN python -m pip install pip --upgrade

# 도커 실행 시 적용할 환경변수
ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

# Host(local)에서 copy 할 디렉토리
ADD . /doivus
# 도커 실행 시 처음 진입할 디렉토리(작업공간)
WORKDIR /doivus

RUN python -m pip install -r requirements.txt

# 도커 실행 시 자동으로 실행되는 명령어
CMD FLASK_APP=app.py flask run -h 0.0.0.0 -p 5000 

# Docker 빌드하기
# Dockerfile이 있는 위치에서 다음과 같은 명령어 수행
docker build -t <repo>:<tag> .
```

### Upload

```bash
# upload를 위해 생성한 이미지 tag 달기 
docker tag <Image-ID> <Hub-ID>/<Repo-Name>:<Tag>

# upload하기
docker push <Hub-ID>/<Repo-Name>:<Tag>
```

### Download

```bash
docker pull <Hub-ID>/<Repo-Name>:<Tag>
```

### Docker 개인 저장소 구성하기

- [https://www.44bits.io/ko/post/easy-deploy-with-docker](https://www.44bits.io/ko/post/easy-deploy-with-docker)
- [https://m-learn.tistory.com/7](https://m-learn.tistory.com/7)

- [http://www.kwangsiklee.com/2017/08/사내-docker-저장소registry-구축하기/](http://www.kwangsiklee.com/2017/08/%EC%82%AC%EB%82%B4-docker-%EC%A0%80%EC%9E%A5%EC%86%8Cregistry-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0/)
- [https://smoh.tistory.com/220](https://smoh.tistory.com/220)
- [https://judo0179.tistory.com/32](https://judo0179.tistory.com/32)
- [https://nirsa.tistory.com/74](https://nirsa.tistory.com/74)
- [https://m.blog.naver.com/complusblog/221000797682](https://m.blog.naver.com/complusblog/221000797682)


# Flask + Pytorch development
### 환경 설정

```bash
# 필요한 애들 설치
pip install flask pytorch torchvision ....
```

### [app.py](http://app.py) 작성 (예시)

```bash
import io
import numpy as np
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/predict', methods=['POST'])
def predict():
    if request.method == 'POST':
        if request.files.get('image'):
            file = request.files['image']

            image_bytes = file.read()
            output = api.prediction(image_bytes)
            res = api.postprocessing(output)

            return jsonify(res)
        else:
            return jsonify({
                'ImageBytes': "",
                'Mophologies': "",
            })

if __name__ == '__main__':
    app.run()
```

### 테스트

```bash
# 로컬 서버 실행
FLASK_ENV=development FLASK_APP=app.py flask run

# curl로 테스트
# -F는 파일 보낼 때 사용함
curl -X POST http://127.0.0.1/predict -F image=@436.png

# python code로 테스트
# 필수 패키지를 불어옵니다.
import numpy as np
import base64
import cv2
import requests
import matplotlib.pyplot as plt

def b64_to_img(im_b64):
    im_bytes = base64.b64decode(im_b64)
    im_arr = np.frombuffer(im_bytes, dtype=np.uint8)
    img = cv2.imdecode(im_arr, flags=cv2.IMREAD_COLOR)
    return img

# Keras REST API 엔드포인트의 URL를 입력 이미지 경로와 같이 초기화 합니다.
KERAS_REST_API_URL = "http://localhost:5000/predict"
BASE_PATH = '/mnt/bgpark/projects/2.IVUS/code/git+code/summary'
IMAGE_PATH = BASE_PATH + "/2nd prolapse_1024.png"

# 입력 이미지를 불러오고 요청에 맞게 페이로드(payload)를 구성합니다.
image = open(IMAGE_PATH, "rb").read()
payload = {"image": image}
 
# 요청을 보냅니다.
r = requests.post(KERAS_REST_API_URL, files=payload).json()
out = r['ImageBytes']
out = b64_to_img(out)

fig = plt.figure(figsize=(16,16))

print(out.shape)
print(out.max())
plt.imshow(out/out.max() * 255)

morph = r['Mophologies']
print(morph)
```

# References

- [https://tutorials.pytorch.kr/intermediate/flask_rest_api_tutorial.html](https://tutorials.pytorch.kr/intermediate/flask_rest_api_tutorial.html)
- [https://github.com/avinassh/pytorch-flask-api-heroku](https://github.com/avinassh/pytorch-flask-api-heroku)
- [https://keraskorea.github.io/posts/2018-10-27-Keras 모델을 REST API로 배포해보기/](https://keraskorea.github.io/posts/2018-10-27-Keras%20%EB%AA%A8%EB%8D%B8%EC%9D%84%20REST%20API%EB%A1%9C%20%EB%B0%B0%ED%8F%AC%ED%95%B4%EB%B3%B4%EA%B8%B0/)
- [https://www.popit.kr/개발자가-처음-docker-접할때-오는-멘붕-몇가지/](https://www.popit.kr/%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80-%EC%B2%98%EC%9D%8C-docker-%EC%A0%91%ED%95%A0%EB%95%8C-%EC%98%A4%EB%8A%94-%EB%A9%98%EB%B6%95-%EB%AA%87%EA%B0%80%EC%A7%80/)
- [https://flask.palletsprojects.com/en/1.1.x/tutorial/deploy/](https://flask.palletsprojects.com/en/1.1.x/tutorial/deploy/)
- [https://yvvyoon.github.io/python/deployment-flask-app-with-docker/](https://yvvyoon.github.io/python/deployment-flask-app-with-docker/)
- [https://jybaek.tistory.com/791](https://jybaek.tistory.com/791)