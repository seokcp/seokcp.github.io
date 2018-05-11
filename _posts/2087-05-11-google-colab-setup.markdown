---
layout: post
comments: true
title:  "Colab Settings"
date: 2018-05-11
categories: Colab Installation
---

Colab(=Colaboratory) 은 Google 에서 만든 무료 클라우드 기반의 주피터 노트북이라고 생각하면 됩니다. 기본적으로 Tesla K80 GPU 를 지원한다고 하네요. Colab 설치 자체는 상당히 간단해서 [fuat](https://medium.com/deep-learning-turkey/google-colab-free-gpu-tutorial-e113627b9f5d){:target="_blank"}의 블로그를 따라가면 간단히  주피터 노트북을 열수 있습니다.

이 글에서 PyTorch, Keras 등 기본으로 설치 되어 있지않은 라이브러리들을 (TF는 당연히 설치 되어있어요) 설치하는 방법과 구글 드라이브 마운팅 하는 방법등을 알아 볼려고 합니다.

확실치는 않지만 어떤 이유에서든 노트북의 세팅이 주기적으로(?) 리셋이 되는 것 같습니다. 그래서 오래 사용하지 않으면 아래의 프로세서를 반복해야 하는 상황이 발생합니다.

* * *

#### 1. **PyTorch** 설치하기

먼저 tensorflow 가 설치 되어 있는지 확인해 보도록 하겠습니다. 1.7.0 이 설치 되어있다는 메세지가 나옵니다.
```python
import tensorflow as tf
print(tf.__version__)
```
*1.7.0*

다음 `import torch` 를 실행해 보면 다음과 같은 에러가 발생하게 됩니다.

```python
---------------------------------------------------------------------------
ModuleNotFoundError                       Traceback (most recent call last)
<ipython-input-6-eb42ca6e4af3> in <module>()
----> 1 import torch

ModuleNotFoundError: No module named 'torch'

---------------------------------------------------------------------------
NOTE: If your import is failing due to a missing package, you can
manually install dependencies using either !pip or !apt.

To install torch, click the button below.
---------------------------------------------------------------------------
```

위에서 알려준대로 **install torch** 버튼을 클릭해서 다음의 명령을 실행합니다 (insert 링크를 클릭하면 자동으로 cell로 복사 됩니다.)

```python
# http://pytorch.org/
from os import path
from wheel.pep425tags import get_abbr_impl, get_impl_ver, get_abi_tag
platform = '{}{}-{}'.format(get_abbr_impl(), get_impl_ver(), get_abi_tag())

accelerator = 'cu80' if path.exists('/opt/bin/nvidia-smi') else 'cpu'

!pip install -q http://download.pytorch.org/whl/{accelerator}/torch-0.3.0.post4-{platform}-linux_x86_64.whl torchvision
import torch
```

버젼을 확인해 보니 아래와 같습니다.

```python
print(torch.__version__)
```
*0.3.0.post4*

참고로 Python 버젼을 노트북상에서 다음과 같이 확인 가능합니다.

```python
import sys
print(sys.version_info)
```
#### 2. **Google Drive** 마운트 하기

구글 드라이브 마운트를 위해서는 다음 명령들을 실행시켜야 하는데 몇 번의 authorization 과정을 (암호입력) 요구합니다.

```bash
!apt-get install -y -qq software-properties-common python-software-properties module-init-tools
!add-apt-repository -y ppa:alessandro-strada/ppa 2>&1 > /dev/null
!apt-get update -qq 2>&1 > /dev/null
!apt-get -y install -qq google-drive-ocamlfuse fuse
from google.colab import auth
auth.authenticate_user()
from oauth2client.client import GoogleCredentials
creds = GoogleCredentials.get_application_default()
import getpass
!google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret} < /dev/null 2>&1 | grep URL
vcode = getpass.getpass()
!echo {vcode} | google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret}
```

이유는 잘 모르겠지만 디렉토리 이동을 위해서는 다음과 같이 `chdir()` 명령어를 사용해야 합니다. `!ls -a` 또는 `!pwd` 같은 명령어는 동작하는데 `!cd` 명령은 동작하지 않습니다. 여기에 대한 자세한 설명은 [The exclamation point](https://github.com/sr320/course-fish546-2016/issues/28){:target="_blank"} 를 참조하세요.

```python
import os
os.chdir("/home")
```

아래 명령을 실행시키면 최종적으로 `/home/drive`를 만들고 자신의 구글 드라이브와 마운트하게 됩니다.

```bash
!mkdir -p drive
!google-drive-ocamlfuse drive
```

#### 3. 설치된 라이브러리 확인하기

새로운 노트북을 열때 마다 기존 노트북에서 세팅해 놓았던 환경들이 다 날아가 버리는 것처럼 보이는데 어떻게 자동적으로 환경을 올리는지를 몰라서 (가능한지도 모르겠음) 머신러닝에 필요하다가 생각되는 라이브러리를 체크해 주고 설치 되었으면 올려주는 간단한 파이썬 스크립트를 만들어 보았습니다.

```python
libnames = ['scipy', 'numpy', 'matplotlib', 'pandas', 'sklearn', 'pydotplus', 'h5py','tensorflow','keras', 'pytorch']

for libname in libnames:
  try:
    lib = __import__(libname)
  except ImportError:
    print(libname + " is not installed")
  else:
    print(libname + " " + lib.__version__)
```

새로운 colab 노트북을 열자마자 스크립트를 실행해 본 화면 입니다.

![라이브러리 체크](https://seokcp.github.io/images/colab_lib_list.png)