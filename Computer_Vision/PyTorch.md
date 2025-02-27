## 파이토치 특징
#### 동적 계산 그래프 
- TensorFlow의 초기 버전과 달리, PyTorch는 실행 시점에서 계산 그래프를 통적으로 생성
- 코드 직관적. 디버깅 쉬움. 실행 중에 그래프 변경 가능

#### 자동 미분 (Autograd)
- PyTorch의 `autograd` 모듈은 자동으로 그래디언트를 계산하여 역전파 수행
- 코드 내에서 간단하게 `.backward()`를 호출하면 미분 자동으로 계산

#### 텐서(Tensor) 연산 및 GPU 가속 지원
- PyTorch의 `Tensor`는 `NumPy`의 배열과 유사하지만, GPU 연산을 지원하여 학습 속도를 크게 향상시킴
- CUDA를 사용하여 GPU 연산을 쉽게 적용 가능


## 파이토치 신경망 기본 구성
1. 모듈 클래스를 이용해 신경망 만들기
2. 데이터셋 클래스를 이용해 데이터를 불러와 학습


### 데이터 호출
- **`__init__`** : 데이터셋을 초기화하고, 필요한 파일을 로드하거나 경로 설정
- **`__len__`** : 데이터셋의 전체 크기(데이터 개수) 반환
- **`__getitem__(self, index)`** : 인덱스에 해당하는 데이터 (정답) 반환

-  `DataLoader`
	 `torch.utils.data.DataLoader`는 미니배치(batch) 단위로 데이터를 불러오는 역할

```python
#### 1. 데이터셋 로드
## 이미지 데이터 셋 클래스

import torch
import os
from torchvision import transforms
from PIL import Image
from torch.utils.data import Dataset, DataLoader

class CustomImageDataset(Dataset):
    def __init__(self, image_dir, transform=None):
        self.image_dir = image_dir
        self.transform = transform
        self.image_files = os.listdir(image_dir)

    def __len__(self):
        return len(self.image_files)

    def __getitem__(self, idx):
        img_path = os.path.join(self.image_dir, self.image_files[idx])
        image = Image.open(img_path).convert("RGB")  # 이미지를 RGB 모드로 로드

        if self.transform:
            image = self.transform(image)
        
        label = 1 if "dog" in self.image_files[idx] else 0  
        # 파일 이름에 "dog"가 있으면 1, 아니면 0

        return image, torch.tensor(label, dtype=torch.long)

# 이미지 변환 정의
transform = transforms.Compose([
    transforms.Resize((128, 128)),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.5, 0.5, 0.5], std=[0.5, 0.5, 0.5])
])

# 데이터셋 및 데이터로더 생성
dataset = CustomImageDataset(image_dir="path/to/images", transform=transform)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

# 데이터 확인
for images, labels in dataloader:
    print(images.shape, labels.shape)  # (배치 크기, 채널, 높이, 너비), (배치 크기,)
    break

```


### 신경망 모델 정의
- `__init__(self)` (생성자) : 모델의 구성요소 정의
- `forward(self, x)` (순전파) : 신경망 동작 정의

```python

#### 2. 신경망 모델 정의
import torch

class NeuralNet(nn.Module): 
	def __init__(self): 
		super(NeuralNet, self).__init__() 
		self.fc1 = nn.Linear(28 * 28, 128) 
		self.fc2 = nn.Linear(128, 64) 
		self.fc3 = nn.Linear(64, 10) 
        
    def forward(self, x): 
	    x = x.view(-1, 28 * 28) # Flatten the input 
	    x = F.relu(self.fc1(x)) 
	    x = F.relu(self.fc2(x)) 
	    x = self.fc3(x) 
	    # No activation function before softmax (included in loss) 
	    return x
```

### 신경망 학습

- `zero_grad()` : 역전파 수행하기 전에 기존의 기울기 초기화
- `forward()`  : 모델에 입력 데이터를 전달하여 예측값 생성
- `criterion()` : 예측값과 실제값 사이의 loss 계산
- `backward()` : `loss.backward()`를 호출하면 PyTorch의 자동 미분(Autograd) 기능이 활성화되며, 각 파라미터에 대한 그래디언트가 계산됨
- `step()` : 옵티마이저가 역전파로 계산된 gradient를 사용하여 모델의 가중치 업데이트

```python
#### 3. 모델, 손실함수, 옵티마이저 설정
device = torch.device("cuda" if torch.cuda.is_available() else "cpu") 
model = NeuralNet().to(device) 
criterion = nn.CrossEntropyLoss() 
optimizer = optim.Adam(model.parameters(), lr=0.001) 

#### 4. 학습 루프 
epochs = 5 
for epoch in range(epochs): 
	model.train() # 학습 모드 설정 
	running_loss = 0.0 
	
	for images, labels in train_loader: 
		images, labels = images.to(device), labels.to(device) 
		
		optimizer.zero_grad() # 기울기 초기화 
		outputs = model(images.view(-1, 28*28)) # 순전파 
		loss = criterion(outputs, labels) # 손실 계산 
		loss.backward() # 역전파 
		optimizer.step() # 가중치 업데이트 
		
		running_loss += loss.item() 
         
		print(f"Epoch {epoch+1}/{epochs}
			,Loss: running_loss/len(train_loader):.4f}")
```

### 모델 평가
- `eval()` : 드롭아웃과 배치 정규화 등의 레이어가 평가모드로 전환됨
- `torch.no_grad()` : 평가 단계에서는 gradient 계산 필요 없으므로 `torch.no_grad()` 사용하여 연산 속도를 높이고 메모리 절약
- `torch.max()` : 모델의 예측값 중 가장 높은 확률을 가진 클래스 찾음

```python
#### 5. 모델 평가

model.eval() 
correct = 0 
total = 0 

with torch.no_grad(): 
	for images, labels in test_loader: 
		images, labels = images.to(device), labels.to(device) \
		outputs = model(images.view(-1, 28*28)) 
		_, predicted = torch.max(outputs, 1) 
		total += labels.size(0) 
		correct += (predicted == labels).sum().item() 

print(f"Test Accuracy: {100 * correct / total:.2f}%")
```