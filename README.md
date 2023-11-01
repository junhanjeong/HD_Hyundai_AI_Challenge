# 🚢HD현대 AI Challenge (정형 회귀모델)   
[인하대 X.ai Team]  
  [ColdTbrew](https://github.com/ColdTbrew)  
  [junhanjeong](https://github.com/hyjk826)  
  [Claude-AD](https://github.com/Claude-AD)  

## 대회 사이트
   [데이콘]( https://dacon.io/competitions/official/236158/overview/description)

## 최종 선택전략
다양한 전처리 방식(결측치 처리, 스케일링, 피처 추가) 등을 진행하였지만 기본 데이터에 autogluon으로 학습시키고 ATA와 CI_HOUR를 더한 Bert_time을 이용하여 후처리해주는 것이 가장 mae가 낮게 나왔기 때문에 autogluon으로 학습 후 Bert_time을 이용한 후처리 방식을 사용하게 되었습니다.

# How to get final csv file (1번부터 3번까지 순서대로 실행하면 됩니다.) 📌

## 0. 구성 양식
data 폴더: train.csv, test.csv, sample_submission.csv가 있습니다.
out_csvs 폴더: 3개의 ipynb로부터 예측값, bert_time을 이용한 CI_HOUR 가능값 리스트 등 만들어지는 csv파일들이 저장될 곳입니다.
saved_model: README 파일을 작성해놓았는데 저희가 미리 학습시켜놓은 autogluon 모델을 다운받을 수 있는 주소를 적어두었습니다. 용량이 너무 큰 관계로 받을 수 있는 링크를 포함함을 알려드립니다.
3개의 ipynb파일: 최종파일을 도출하기 위한 3개의 노트북 파일입니다.  

- 0. model_load_infer.ipynb : private score 복원을 위한 모델 로드와 predict과정 노트북 파일입니다. 1번파일의 결과값과 동일합니다.
- 1. train_auto6.ipynb: autogluon을 통해 회귀모델이 예측한 CI_HOUR을 csv로 저장하는 코드입니다.
- 2. make_use_bertTime_and_get_CI_HOURS.ipynb: train데이터셋에서 bert_time을 구하고 train의 기후열들과 똑같은 test데이터셋을 찾아 가능한 CI_HOUR의 값의 리스트를 각 값마다 저장하는 csv파일을 만드는 코드입니다.
- 3. make_final_auto06.ipynb: 위의 두 파일을 이용하여 기존 회귀모델이 예측한 값과 Bert_time을 이용해 구한 CI_HOUR 가능 리스트들을 적절히 비교하여 하나의 CI_HOUR로 선택하여 적용해서 최종 csv파일을 만들어주는 코드입니다.

## 1. start training (train_auto06.ipynb)
train_auto06.ipynb 파일을 실행시키면 autogluon 모델을 학습시키고 test파일에 대해 predict하여 회귀모델이 예측한 값을 auto06.csv파일로 저장합니다.
(autogluon 모델을 돌리는데 약 5시간이 소요되기 때문에, saved_model 폴더에 model을 다운받는 링크를 저장해두었습니다.)

## 2. train셋의 Bert_time을 이용하여 test셋의 가능 CI_HOUR 도출 (make_use_bertTime_and_get_CI_HOURS.ipynb)
1) train셋에서 Bert_time 열을 만들고 필요한 열만 추출하여 small_train 데이터프레임을 만듭니다.
2) 병렬처리를 이용해 train셋의 기후컬럼 6열과 test셋의 기후컬럼 6열이 일치하면 test셋에서 (train의 Bert_time 값 - test셋의 ATA 값)을 적용해 CI_HOUR를 계산하여 리스트에 저장합니다.
이 때, 리스트에 저장되는 가능한 CI_HOUR 값은 여러개가 될 수 있고 없을 경우에 -1을 저장합니다. 또한, 값이 음수가 되는 것은 저장하지 않습니다.
3) 완료되면 use_bertTime_and_get_CI_HOURS.csv 파일로 저장합니다.

## 3. 회귀모델의 값과 Bert_time으로 구한 CI_HOUR 가능값들을 비교하여 최종 CI_HOUR csv파일 생성 (make_final_auto06.ipynb)
1) a는 Bert_time으로 구한 가능한 CI_HOUR 리스트값, b는 회귀모델로 예측한 CI_HOUR 값으로 설정합니다.
2) a값이 -1이면 회귀모델이 예측한 값을 적용합니다.
3) a값이 1개 이상이면 회귀모델이 예측한 값과 가장 가까운 값 closest_value를 구합니다.
4) closest_value와 회귀모델이 예측한 값의 차이 > 회귀모델이 예측한 값 * 71일 경우 회귀모델 값을 적용합니다. (차이가 너무 심할 경우)
5) cloest_value가 2200 이상일 경우 회귀모델 값을 적용합니다. (train셋에서 CI_HOUR 최대값이 2159였으므로 2200 넘는 것을 이상치로 취급)
6) 그 외에는 Bert_time으로 구한 closest_value를 적용합니다.
7) 최종파일(final_auto06.csv)를 생성합니다.

## 4. 라이브러리 환경
- requirements.txt

