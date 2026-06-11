# 실제 블랙박스 환경을 위한 YOLOv11 기반 도로 위험물 탐지 및 전처리 한계 분석

## 프로젝트 소개 
본 프로젝트는 YOLOv11 모델을 활용하여 도로 위 치명적인 사고를 유발하는 위험물(포트홀, 균열, 맨홀 등)을 탐지하는 시스템을 구축하고, **학습 데이터(고화질/정면)와 실제 환경(블랙박스 노이즈/광각) 간의 데이터 환경 차이(Domain Gap)에 따른 이미지 전처리 기법의 한계를 정량적·정성적으로 분석**한 연구입니다.

## 주요 연구
단순히 AI 모델의 성능을 높이는 것을 넘어, 보편적으로 사용되는 이미지 전처리 기법들이 블랙박스라는 특정 실환경 도메인에서 모델의 판단력을 어떻게 훼손하는지 분석했습니다.

1. **CLAHE(명암 강조)의 역효과:** 인위적인 명암 강조는 아스팔트 질감과 미세 노이즈까지 날카롭게 증폭시켜 오탐지(False Positive)를 급증시킴.
2. **ROI Crop의 한계:** 불필요한 배경을 잘라낼 경우, 모델이 도로의 원근감과 주변을 파악할 '공간적 맥락(Context)'을 상실하여 탐지 성능이 저하됨.
3. **정성적 평가(Inference) 전환:** 이종 데이터셋 간 YAML 클래스 구조 불일치 문제를 식별하고, 수치적 오류를 방지하기 위해 실제 주행 영상 기반의 바운딩 박스 매핑(정성적 평가)으로 검증 전략을 수정함.

![framework](Project%20Flowchart.png)
<p align="center"><b>본 연구의 수행 절차 흐름도 (데이터 전처리 및 YOLOv11 기반 평가 프로세스)</b></p>

## Installation

To get started, clone the repository and install dependencies:

```bash
!git clone https://github.com/AI-CML/AI-Project.git
%cd AI-Project
!pip install -r requirements.txt
```

## Requirements

```text
ultralytics
opencv-python
matplotlib
numpy
```

## Datasets

For the `Road Damage Dataset` , you can download the original dataset from [Kaggle](https://www.kaggle.com/datasets/lorenzoarcioni/road-damage-dataset-potholes-cracks-and-manholes).

---

## 실험 결과 (Experimental Results - Val 기준)
해외 오픈 데이터셋 환경에서 다양한 기법을 적용하며 성능 변화를 측정했습니다.

> **💡 Note: Test 평가 대신 Val 지표를 사용한 이유**
> 학습 데이터와 직접 수집한 실전 테스트 데이터 간의 YAML 클래스 ID 구조 불일치(Label Mismatch)가 발생했습니다. 강제 레이블 변환 시 발생할 수 있는 평가 지표의 왜곡을 방지하기 위해, 정량적 수치(mAP) 평가는 **Val 데이터셋**을 기준으로 통일했습니다. 대신, Test 데이터는 수치화에 얽매이지 않고 실제 주행 영상에서 바운딩 박스가 정확히 매핑되는지 확인하는 **정성적 검증(Qualitative Evaluation)** 으로 전환하여 실효성을 입증했습니다.

| 실험 및 적용 기법 | mAP | 결과 분석 |
| :--- | :--- | :--- |
| **Baseline (YOLO11s)** | **0.557** | **해외 데이터로 학습된 순수 탐지 성능 (가장 높음)** |
| Model Upgrade (YOLO11m) | 0.557 | 모델 체급을 올렸으나 동일한 수치 도출 (데이터 병목 확인) |
| Augmentation (데이터 증강) | 0.543 | 인위적인 증강 기법이 오히려 노이즈를 증폭시켜 성능 저하 |
| Processing (ROI Crop) | 0.535 | 불필요한 배경 제거가 도로 원근감 및 주변 탐지 능력 훼손 |
| Processing (CLAHE) | 0.477 | 명암 강조로 아스팔트 질감/미세 노이즈가 증폭되어 오탐지 급증 |

> **🔥 결론:** 모델 성능을 올리기 위한 인위적인 전처리(ROI, CLAHE, Augmentation)가 실환경에서는 오히려 노이즈를 증폭시키고 맥락을 훼손함. 섣부른 전처리보다 **국내 도로 규격에 맞춘 데이터 구축 등 '데이터 중심(Data-centric)' 접근이 필수적**임을 확인.

---
