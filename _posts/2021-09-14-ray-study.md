---
title: "[Study] Ray"
categories:
  - Ray
tags:
  - ray
  - distributed learning
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---
Date: September 9, 2021 → September 14, 2021

### Ray

- Python에서 쉽게 Multiprocessing을 하게 해주는 package
- 딥러닝/머신러닝에 적용하기 위해 만들어짐
- 효율적인 분산 학습/처리를 위해 단순하고 범용성 높은 API들로 구성됨
- Ray Eco system
    - Native : Ray tune, Ray sgd, etc..
    - third party : pytorch, tensorflow, mlflow, Horovod, etc..
- 장점
    - 기존 코드에서 데코레이션 추가로 간단하게 병렬 처리로 변환
    - 클러스터 환경에서도 사용 가능
    - Dashboard로 process들의 현황 파악 가능
    - 기존 Multi-processing보다 빠름
    - 머신러닝/딥러닝에 최적화
    - 서드파티 등으로 풍부한 기능 연계 가능
    - 중첩 실행 가능

구성요소

- Actor
    - Stateful, class와 같은 역할
- Task
    - Stateless, function(method)와 같은 역할
- Actor, Task 모두 remote 함수로 실행가능 (worker)
    - 실행 시, Future라는 object를 return
    - Task 기본 예제

    ```jsx
    import ray
    # 기본 사용 예제

    @ray.remote -> Task로 선언
    def add(a, filter):
    	return a * filter

    data = np.zeros((N,N))
    # 병렬처리에서 공동으로 사용될 데이터(변경 X)
    # In-memory storage (like shared memory)
    img_id = ray.put(data)

    # .remote(args) -> worker로의 변환 (멀티 프로세싱을 하기 위해)
    # ray.get -> 병렬처리 결과를 리턴받음
    result = ray.get([add.remote(img_id, filters[i]) for i in range(M)])

    # ray.wait 예제
    # 방대한 corpus내 keyword가 포함된 열 찾기
    ray.init(num_cpus=num_cpus, ignore_reinit_error=True)

    @ray.remote
    def check_isin_kwd(df_corpus, kwd):
      return [kwd, df_corpus.str.contains(kwd, regex=False)]

    corpus_id = ray.put(df_data['document'])
    check_isin_kwd_ids = []
    for i in range(len(list_kwd)):
      check_isin_kwd_ids.append(check_isin_kwd.remote(corpus_id, list_kwd[i]))

    # 끝난 task와 아닌 task를 구분하여 처리함 (약 20% 성능 향상)
    while len(check_isin_kwd_ids):
      done_id, check_isin_kwd_ids = ray.wait(check_isin_kwd_ids)
      kwd, ser = ray.get(done_id[0])
      df_result[kwd] = ser
    ```

    - Actor를 이용한 pytorch 예제

    ```jsx
    # condition
    # 데이터는 list 형태로 축적
    # gpu 1개분 처리량 보다 커지면 추가 gpu를 할당하여 사용
    # gpu가 덜 필요하면 할당한 gpu release

    os.environ['CUDA_VISIBLE_DEVICES'] = '5,6'

    # ray.init을 하면 driver 실행?
    # driver는 프로그램의 메인 루트
    # init에 별도의 argument를 넣지 않을 시, 모든 cpu를 사용함
    ray.init(num_cpus=8, num_gpus=2)

    @ray.remote(num_gpus=1) -> 이렇게만 해주어도 actor로써 기능함
    class GPUActor(object):
    	def __init__...

    	def release_cuda_memory(self):
        # [중요] pytorch에서 점유하고 있던 해당 gpu 메모리(캐시된 메모리)를 
        # 해제하는 기능을 아래 방법으로 제공함
        torch.cuda.empty_cache()
        return 1
        
      def print_gpu_ids(self):
        # gpu id 확인용 테스트 코드
        return "This actor is allowed to use GPUs {}.".format(ray.get_gpu_ids())

    actors = [GPUActor.remote() for _ in range(2)]

    print(ray.get([act.print_gpu_ids.remote() for act in actors]))

    preprocessed_ids = [actor.preprocess.remote(list_filepath[batchsize*i:batchsize*(i+1)]) for i, actor in enumerate(actors)]
    pred_ids = [actor.predict.remote(preprocessed_id) for preprocessed_id, actor in zip(preprocessed_ids, actors)]

    ray.get(actors[1].release_cuda_memory.remote())
    ```

    - 일반적으로 lifetime을 지정하지 않으면 actor는 driver가 끝날 때까지 지속

    ```jsx
    # Job과 Actor를 분리하여 사용
    # detached는 Job exited되어도 actor가 살아 있음
    actor1 = Actor.options(name="CounterActor", lifetime="detached").remote()

    # 나중에 이런식으로 재호출 가능
    actor1 = ray.get_actor("CounterActor")
    print(ray.get(actor1.get_counter.remote()))

    # 일반적으로 job 종료와 함께 actor도 종료됨
    # 하지만, 예외적으로 수동으로 조절이 필요한 경우
    # actor handle(actor id) 가 범위가 벗어난 경우
    actor = Actor.remote() 후 actor = 'others'

    # 강제종료가 필요한 경우
    ray.actor.exit_actor()
    ray.kill(actor_id)

    # 수동 종료 등을 수행 할 때는 gpu를 사용하고 있다면, gpu를 먼저 release를 해주는게 좋음
    ```

### Ray Tune

- Hyperparameter Optimization Library
- High-level api로 구성되어 있어 사용법이 간단함

```jsx
from ray import tune

def objective(a, b):
	return a**2+b**3-(a/b)*0.5 ....

def training_func(config):
	# TODO
	# score = objective(1,3)

# 주기적으로 업데이트를 해주어야 함
tune.report(mean_loss=score)

result = tune.run(
	training_func, # 탐색을 하는 함수 -> run 시, Trial라는 객체로써 동작
	config={ # 탐색 함수의 인자 값
}
)
```

### RaySGD

- Distributed Deep Learning을 위한 Library
- Multi GPU/Node 학습도 모두 지원함
- TrainingOperator라는 추상클래스를 정의하여 학습/검증 진행 가능

```jsx
class Example(TrainingOperator):
	def setup(self, config):
		model, loss, optimizer, ...
		self.model, self.optimizer = self.register(
			models=model,
			optimizers=optimizer,
			criterion=loss,
		)
		self.register_data(
			train_loader=DataLoader(),
			validation_loader=DataLoader(),
		)

# 기본적으로 다수의 gpu를 사용하게 CUDA_VISIBLE_DEVICES를 설정하면,
# DDP를 이용하여 학습함. wrap_ddp 옵션을 통해 DDP mode를 끌 수도 있음
trainer = TorchTrainer(
	training_operator_cls=Example,
	config={},
	use_gpu=True,
)

for epoch in range():
	trainer.train()

# 만약에 TrainingOperator로 학습한 모델의 HPO를 하고 싶다면
# .as_trainable() 함수로 쉽게 넘길 수 있는 객체 생성 가능

# PytorchLightning으로 개발한 모델을 사용할 때
TrainingOperator.from_ptl(lightning_obj)
```

References

- [https://github.com/ray-project/ray](https://github.com/ray-project/ray)
- [https://zzsza.github.io/mlops/2021/01/03/python-ray/](https://zzsza.github.io/mlops/2021/01/03/python-ray/)
- [https://medium.com/naver-shopping-dev/ray-로-pytorch-model-inference-하기-77ce11304604](https://medium.com/naver-shopping-dev/ray-%EB%A1%9C-pytorch-model-inference-%ED%95%98%EA%B8%B0-77ce11304604)
- [https://riiidtechblog.medium.com/ray-확장-가능한-고성능-분산-병렬-machine-learning-프레임워크-f17f9c9cbef3](https://riiidtechblog.medium.com/ray-%ED%99%95%EC%9E%A5-%EA%B0%80%EB%8A%A5%ED%95%9C-%EA%B3%A0%EC%84%B1%EB%8A%A5-%EB%B6%84%EC%82%B0-%EB%B3%91%EB%A0%AC-machine-learning-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC-f17f9c9cbef3)