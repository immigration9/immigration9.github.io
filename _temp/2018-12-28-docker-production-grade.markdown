## Production Grade Flow
* development -> testing -> deployment 의 과정

* Github Repo
  * Feature branch: 개발 서버
  * Master branch: 운용 서버

Feature branch로 Push를 하고, Pull Request를 통해 master로 merge가 된다.
master는 Travis CI로 연결된다.
결과가 통과면 AWS에 호스팅된다.

## 정리
1. Dev Phase
- 기능 추가/수정
- master가 아닌 branch에 변경 사항 기입

2. Github로 Push
3. master로 merge하기 위해 Pull Request 생성

4. Test Phase
- Travis CI로 코드 Push
- Test Run

5. Pull Request를 master와 merge

6. Prod Phase
- Travis CI로 코드 Push
- Test Run
- AWS에 배포 (Elastic Beanstalk)

## React Project 생성
`create-react-app frontend`

`Dockerfile.dev` 파일 생성
```Dockerfile
FROM node:alpine

WORKDIR '/app'

COPY package.json ./
RUN npm install

COPY ./ ./

CMD ["npm", "run", "start"]
```
custom file사용하기: `docker build -f Dockerfile.dev .`

* `/node_modules` 폴더는 제거한 후에 `.Dockerignore` 파일에 추가하자

## 파일 변경사항 감지하기
Docker의 Volume을 사용하자
`docker run -v $(pwd):/app (image id)`
