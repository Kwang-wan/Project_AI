# Tensorflow 2 버전으로 다시 만든 MuseGAN



**1. Generator와 Discriminator**

"미술관에 GAN 딥러닝 실전 프로젝트"(한빛미디어/데이비드 포스터 지음/ 박해선 옮김) 교재의 코드를 참고하였습니다.

다만, 교재에서 화음/스타일/멜로디/리듬에 대해 각각 잡음을 입력 받도록 만들어져 있는 부분은 변경했습니다.

하나의 잡음을 입력 받고 이 잡음을 각각 화음/스타일/멜로디/리듬 부분으로 람다 레이어를 통해 분배하는 형식으로 이루어졌습니다. 잡음의 수준을 낮추어 생성된 음악의 바리에이션을 줄이는 대신, GAN의 학습을 용이하게 하여 MuseGAN이 프로젝트에 적합한지 테스트해보려는 목적이었습니다.



**2. WGAN-GP 프레임워크**

교재에서는 Tensorflow 1.x 버전을 사용하는데, Tensorflow 2.x에서는 _Merge를 임포트할 수 없는 문제가 발생하여 GAN 프레임워크가 실행되지 않았습니다.

MuseGAN에서는 WGAN-GP 프레임워크를 사용하므로, Keras 사이트의 Code Example 중 이미지 처리에 대한 WGAN-GP 예제를 참고하여 MuseGAN에 적용했습니다.

(사이트 링크 https://keras.io/examples/generative/wgan_gp/)



**3. 데이터셋**

> 사용된 데이터는 프로젝트 평가만을 위해 제출되며, 깃허브에는 올리지 않을 예정입니다.
>
> 필요하신 분은 사이트 링크를 참고하세요.

최종적으로는 깃허브(사이트 링크 https://github.com/czhuang/JSB-Chorales-dataset/blob/master/Jsb16thSeparated.npz)에 공개된 4성부로 이루어진 바흐 코랄 음악 데이터셋만을 사용하였습니다.

새로운 데이터셋을 만드는 시도를 해보았는데, 이는 '5. 시행착오'에서 후술하겠습니다.



**4. 산출물**

MuseGAN 모델에 대한 최종 산출물은 MuseGAN.mid 파일로, MuseGAN을 통해 만들어진 두 마디의 멜로디를 합쳐 1분 가량의 짧은 곡으로 만든 것입니다.

곡의 마지막 마디만, 곡이 끝나는 느낌을 주기 위해 음의 피치를 수정하였습니다.



**5. 시행착오**

바흐 코랄 음악 데이터셋의 경우, 4성부로 이루어진 곡으로 다중 선율의 음악을 만드는 MuseGAN에 적합한 데이터입니다. 하지만 단 2 마디만을 제공하여 음악의 길이가 짧습니다.

MuseGAN 모델은 음악을 이미지로 생각합니다. Discriminator는 진짜 곡(2마디)과 Generator가 생성한 가짜 곡(2마디)의 진위를 판별하도록 학습하므로, Generator 또한 진짜 곡과 같이 2마디의 곡을 생성해야 합니다.

그러므로, 1분 가량의 곡을 만들기 위해 데이터셋을 새로 마련하는 방안을 고려했습니다.

가장 처음엔 음악 데이터 여러 개를 직접 수집해 데이터셋을 만드는 방안을 고려했지만, MuseGAN을 위한 데이터셋에는 일단 두 가지 충족해야 하는 조건이 있습니다.

1. 만약 4성부의 음악을 만들고자 한다면, 모든 데이터가 동일한 4성부로 이루어져야 합니다. 이는 악기의 경우도 마찬가지입니다. 만약 4 종류의 악기를 사용하는 음악이라면, 모든 데이터가 동일한 4 종류의 악기를 사용하는 데이터를 구해야 합니다. 이는 성부와 악기마다 음역대가 서로 다르기 때문입니다.
2. 음악의 길이가 동일해야 합니다. 이는 MuseGAN이 음악을 이미지로 이해하기 때문입니다. 이미지로 예를 들면 이미지를 학습시키기 위해 32*32 픽셀의 동일한 픽셀의 이미지를 데이터로 사용하는 것과 같습니다. 이러한 제약조건이 MuseGAN에서는 마디의 갯수에 해당합니다.

대부분의 음악 데이터에는 저작권이 있으므로 무자본으로 다음의 조건을 모두 충족하는 데이터를 얻는 데에 한계가 있었습니다.

그래서 모색한 새로운 방안은 '이미지 증폭' 방식에 착안해 부족한 음악 데이터를 늘려보는 것이었습니다. 시험해본 방식은 다음과 같습니다.

​	A-1. 음악 데이터를 4 종류의 악기를 사용하는 음악 데이터를 구해 전처리를 거쳐 데이터 하나를 작성하기

​	A-2. 작성한 데이터의 피치를 수정하는 조바꿈을 시도

​	B-1. 두 마디의 바흐 코랄 데이터를 반복하여 마디 갯수를 늘리기

​	C-1. A를 통해 작성한 데이터의 비중과 B 데이터셋의 비중이 비슷하도록 양을 조절해 두 데이터를 합치기

이렇게 만들어진 데이터를 사용하면 A곡을 B곡의 느낌으로 편곡한 느낌의 곡을 생성하지 않을까 기대하였지만, 실제로는 제대로 된 학습이 이루어지지 않았습니다.

(시험해본 데이터는 data 폴더에 lift_every_voice_amplified.npy 파일로 저장해두었습니다.)

위와 같은 시행착오 끝에 MuseGAN을 사용하는 것은 프로젝트의 기간과 자원을 고려하였을 때 현실성이 떨어진다는 판단을 하였고, MuseGAN으로 생성된 두 마디 곡을 이어붙여 1분 가량의 최종 산출물을 만들어내는 것으로 MuseGAN 모델을 마무리하였습니다.