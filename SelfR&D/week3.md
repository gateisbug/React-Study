# 왜 Electron을 써야할까?
Electron은 javascript 프로젝트를 브라우저의 종속성 없이 실행 시키기 위한 프레임워크다. 여기서 중요한 건 브라우저의 종속성이 없다는 것.

즉, URL이나 index.html과 같이 브라우저를 통해 보는 것이 아닌 exe 파일을 실행시켜서 프로젝트를 보기 위해 사용하는 것이다.

이 프레임워크를 처음 사용한 것은, 내가 친구한테서 의뢰를 받아서 였다. 그때 요구사항은 웹 크롤러를 이용해서 미국 etf의 티커를 입력하면 해당되는 페이지로 이동해서 정보를 긁어오는 프로젝트를 만드는 것이 목적이었다. 친구는 개발자가 아니었기에, 누구나 쉽게 사용할 수 있는 exe 파일로 만드는 것이 당연하다고 생각했고, 그렇게 되면서 electron을 사용하게 되었다.

## Electron은 어떻게 동작하는가?
여기서 말하고 싶은 건 우리가 알 필요는 없는 패키징의 이론이 아니라, 대략적으로 우리가 만든 프로젝트를 어떻게 exe 파일로 만드는지에 대해서 말하고자 하려는 것이다.
방법은 아주 간단한데, 우리가 만든 페이지를 electron에 포함시켜서 함께 패키징하면 된다. 정적 배포는 이 과정이 아주 간단하기 때문에 개념만 알면 누구나 해볼 수 있다.

# Usage
1. 패키징할 프론트엔드 페이지 작성
2. electron 라이브러리 설치
3. electron 디렉토리 및 electron.cjs, preload.cjs 생성
4. electron.cjs 구성
5. ipcRenderer 구성
6. preload.cjs 구성
7. electron 패키징

# Electron-Vite
[#Official Page](https://youtu.be/X2Rh0pvgITw?si=DpnBZBg7emm8OqsJ)
- Vite로 만든 프로젝트를 build 후 electron으로 패키징해주는 프레임워크다
- electron 기초작업이 귀찮다면 이걸 쓰자
