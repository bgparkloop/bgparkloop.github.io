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

#### 2.2. 데이터 준비
데이터를 넣어둘 폴더를 생성한다.
- 데이터 폴더 : images
  - images폴더 안에는 train과 test 폴더를 만들어 나중에 필요한 데이터를 넣는다.

1. Tensorflow API에 사용되는 annotation 파일들은 labelImg를 기준으로 작성되므로, 해당 프로그램을 통해 annotation하여 xml파일들을 만든다.
2. 작업한 이미지와 xml파일들을 images폴더에서 train과 test에 적절하게 나눠넣는다.
3. images폴더 밖에서 **xml_to_csv.py** 파일을 이용하여 csv형태로 변환한다.
  ~~~
    import os
    import glob
    import pandas as pd
    import xml.etree.ElementTree as ET
    ​
    def xml_to_csv(path):
        xml_list = []
        for xml_file in glob.glob(path + '/*.xml'):
            tree = ET.parse(xml_file)
            root = tree.getroot()
            for member in root.findall('object'):
                value = (root.find('filename').text,
                        int(root.find('size')[0].text),
                        int(root.find('size')[1].text),
                        member[0].text,
                        int(member[4][0].text),
                        int(member[4][1].text),
                        int(member[4][2].text),
                        int(member[4][3].text)
                        )
                xml_list.append(value)
        column_name = ['filename', 'width', 'height', 'class', 'xmin', 'ymin', 'xmax', 'ymax']
        xml_df = pd.DataFrame(xml_list, columns=column_name)
        return xml_df

    def main():
        for folder in ['train','test']:
            image_path = os.path.join(os.getcwd(), ('images/' + folder))
            xml_df = xml_to_csv(image_path)
            xml_df.to_csv(('images/' + folder + '_labels.csv'), index=None)
            print('Successfully converted xml to csv.')
    ​
    main()
  ~~~
4. images폴더 밖에서 **generate_tfrecord.py** 파일을 이용하여 record형태로 변환한다. 이 때, 코드 내에서 class_text_to_int 함수의 return 부분을 학습하고자 하는 class들로 수정하여야 한다.
  ~~~
    """
    Usage:
    # From tensorflow/models/
    # Create train data:
    python generate_tfrecord.py --csv_input=images/train_labels.csv --image_dir=images/train --output_path=train.record
    ​
    # Create test data:
    python generate_tfrecord.py --csv_input=images/test_labels.csv  --image_dir=images/test --output_path=test.record
    """
    from __future__ import division
    from __future__ import print_function
    from __future__ import absolute_import
    ​
    import os
    import io
    import pandas as pd
    import tensorflow as tf
    ​
    from PIL import Image
    from object_detection.utils import dataset_util
    from collections import namedtuple, OrderedDict
    ​
    flags = tf.app.flags
    flags.DEFINE_string('csv_input', '', 'Path to the CSV input')
    flags.DEFINE_string('image_dir', '', 'Path to the image directory')
    flags.DEFINE_string('output_path', '', 'Path to output TFRecord')
    FLAGS = flags.FLAGS
    ​    ​
    norm_list = ['normal', 'Normal']
    abnorm_list = ['abnormal', Abnormal]
    ​    ​
    # TO-DO replace this with label map
    def class_text_to_int(row_label):
        if row_label in norm_list:
            return 1
        elif row_label in abnorm_list:
            return 2
        else:
            return 3
    ​
    ​
    def split(df, group):
        data = namedtuple('data', ['filename', 'object'])
        gb = df.groupby(group)
        return [data(filename, gb.get_group(x)) for filename, x in zip(gb.groups.keys(), gb.groups)]
    ​
    ​
    def create_tf_example(group, path):
        with tf.gfile.GFile(os.path.join(path, '{}'.format(group.filename)), 'rb') as fid:
            encoded_jpg = fid.read()
        encoded_jpg_io = io.BytesIO(encoded_jpg)
        image = Image.open(encoded_jpg_io)
        width, height = image.size
    ​
        filename = group.filename.encode('utf8')
        image_format = b'jpg'
        xmins = []
        xmaxs = []
        ymins = []
        ymaxs = []
        classes_text = []
        classes = []
    ​
        for index, row in group.object.iterrows():
            xmins.append(row['xmin'] / width)
            xmaxs.append(row['xmax'] / width)
            ymins.append(row['ymin'] / height)
            ymaxs.append(row['ymax'] / height)
            classes_text.append(row['class'].encode('utf8'))
            classes.append(class_text_to_int(row['class']))
    ​
        tf_example = tf.train.Example(features=tf.train.Features(feature={
            'image/height': dataset_util.int64_feature(height),
            'image/width': dataset_util.int64_feature(width),
            'image/filename': dataset_util.bytes_feature(filename),
            'image/source_id': dataset_util.bytes_feature(filename),
            'image/encoded': dataset_util.bytes_feature(encoded_jpg),
            'image/format': dataset_util.bytes_feature(image_format),
            'image/object/bbox/xmin': dataset_util.float_list_feature(xmins),
            'image/object/bbox/xmax': dataset_util.float_list_feature(xmaxs),
            'image/object/bbox/ymin': dataset_util.float_list_feature(ymins),
            'image/object/bbox/ymax': dataset_util.float_list_feature(ymaxs),
            'image/object/class/text': dataset_util.bytes_list_feature(classes_text),
            'image/object/class/label': dataset_util.int64_list_feature(classes),
        }))
        return tf_example
    ​
    ​
    def main(_):
        writer = tf.python_io.TFRecordWriter(FLAGS.output_path)
        path = os.path.join(os.getcwd(), FLAGS.image_dir)
        examples = pd.read_csv(FLAGS.csv_input)
        grouped = split(examples, 'filename')
        for group in grouped:
            tf_example = create_tf_example(group, path)
            writer.write(tf_example.SerializeToString())
    ​
        writer.close()
        output_path = os.path.join(os.getcwd(), FLAGS.output_path)
        print('Successfully created the TFRecords: {}'.format(output_path))
    ​
    ​
    if __name__ == '__main__':
        tf.app.run()
  ~~~

#### 2.3. 설정 준비
학습결과를 저장할 폴더를 생성한다.
- 학습결과 폴더 : training
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

        # label_map_path에는 앞서 만든 labelmap.pbtxt의 경로를 넣어준다.
        # tf_record_input_reader에서 input_path는 뒤에 images에 있는 파일들을 record파일로 변경하여 그 경로를 입력한다.
        train_input_reader {
        label_map_path: "PATH_TO_BE_CONFIGURED/mscoco_label_map.pbtxt"
        tf_record_input_reader {
            input_path: "PATH_TO_BE_CONFIGURED/mscoco_train.record"
        }
        }
        # num_examples는 몇 개의 샘플을 테스트 할 것인지 결정한다. images/test에 있는 이미지 수보다 크면 안된다.
        # max_evals는 최대 테스트할 이미지 수이다.
        # 기본에는 없지만 num_visualizations라는 변수를 추가하면 적혀있는 숫자만큼의 test 이미지에 대한 결과를 tensorboard에서 보여준다.
        eval_config {
        num_examples: 8000
        max_evals: 10
        use_moving_averages: false
        # num_visualizations: 20
        }
        # train_input_read와 같다.
        eval_input_reader {
        label_map_path: "PATH_TO_BE_CONFIGURED/mscoco_label_map.pbtxt"
        shuffle: false
        num_readers: 1
        tf_record_input_reader {
            input_path: "PATH_TO_BE_CONFIGURED/mscoco_val.record"
        }
        }
    ~~~

---
### 3. 학습 진행
#### 3.1. Training
이제 준비는 모두 끝났으니 본격적으로 학습을 진행한다. 아래와 같은 명령어를 입력하여 학습을 진행한다.
- YOUR_API_PATH/models/research/object_detection/legacy/train.py --logtostderr --train_dir=training/ --pipeline_config_path=training/YOUR_CONFIG_FILE

config파일에 지정한 설정대로 학습이 진행되고, training폴더에는 중간중간마다 model checkpoint파일이 저장된다. 학습이 진행되는 그래프를 확인하려면 tensorboard를 설치 후 아래와 같은 명령어로 살펴볼 수 있다.
- tensorboard --logdir=./training

#### 3.2. Evaluation
테스트를 진행하려면, training폴더안에 eval폴더를 만든 후 다음과 같이 명령어를 실행한다.
- YOUR_API_PATH/models/research/object_detection/legacy/eval.py --logtostderr --checkpoint_dir=training/  --pipeline_config_path=training/YOUR_CONFIG_FILE --eval_dir=training/eval

위의 명령어를 실행하면, test폴더에 있는 이미지들로 결과를 내어 각 Class마다 mAP를 계산하여 알려준다. config파일에서 num_visualizations를 추가하였다면 tensorboard에 GRAPH옆에 IMAGE탭이 새로 생기면서 우리가 학습한 모델의 성능을 눈으로 확인이 가능하다.

---
### 4. 번외 임베디드/모바일 환경에서의 딥러닝 모델 사용
Mobile이나 임베디드 환경에서 사용하려면 경량화된 Model로 변경해야한다. Tensorflow API에서 제공하는 export_tflite_xxx_graph.py와 같은 파일로 우리가 학습한 model을 tflite로 변경 가능하다. 하지만, 한 번에 변환되지 않고 ckpt -> pb -> tflite의 변환과정을 거친다.

#### 4.1. pb파일로 변환
아래는 ssdlite_mobilenet에 대한 예제이다. 각자에서 적용할 때는 path부분들을 자신들의 환경에 맞게 변경해야한다. 명령어를 입력하면, tflite_graph.pb와 tflite_graph.pbtxt파일이 생성된다.

~~~
python export_tflite_ssd_graph.py \
--pipeline_config_path=YOUR_TRAINED_MODEL_DIR_PATH/ssdlite_mobilenet_v2_coco.config \
--trained_checkpoint_prefix=YOUR_TRAINED_MODEL_DIR_PATH/model.ckpt-107 \
--output_directory=. \
--add_postprocessing_op=true
~~~

#### 4.2. pb파일을 tflite파일로 변환
변환된 pb파일을 tflite로 변환하기위해서는 bazel설치가 필요하다.
Windows에서 설치는 검색하면 나오므로... Linux 기준으로
- sudo apt-get install bazel=1.0.0

bazel 이용 시 tensorflow가 필요하므로 git으로 다운로드한다.
- git clone https://github.com/tensorflow/tensorflow.git
심지어 tensorflow 버전이 높으면 변환이 어려우니 tensorflow 버전을 1.12.0으로 하면 안정적으로 가능하다.

그 후 tflite로 변환할 pb와 pbtxt파일이 있는 폴더를 OUTPUT_DIR로 지정
- export OUTPUT_DIR=YOUR_PB_DIR

마지막으로, 설치한 bazel을 이용하여 변환작업을 실행한다.
이 때 opt다음에 나오는 tensorflow/lite/toco부분이 우리가 git으로 받은 tensorflow폴더이다.
input_shapes의 경우도 처음 학습할 때 shape과 동일하게 해준다.

- bazel run -c opt tensorflow/lite/toco:toco -- \
 --input_file=$OUTPUT_DIR/tflite_graph.pb \
 --output_file=$OUTPUT_DIR/ssd_mobilenet.tflite \
 --input_shapes=1,300,300,3 \
 --input_arrays=normalized_input_image_tensor \
 --output_arrays='TFLite_Detection_PostProcess','TFLite_Detection_PostProcess:1','TFLite_Detection_PostProcess:2','TFLite_Detection_PostProcess:3' \
 --inference_type=FLOAT \
--allow_custom_ops

> INFO: Build completed successfully, 1 total action 나오면 성공
> Ignoring unsupported type in list attribute with key '_output_types'가 나올 수 있는데 무시하면 됨