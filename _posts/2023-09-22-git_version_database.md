---
layout: post
title:  "Git Version Database"
tags: ["Git","GitHub"]
date:   2023-09-22 11:50:00
---

이전에 'Git&Github'라는 제목으로 Git에 대한 포스트를 쓴 적이 있었고, 해당 포스트에서는 Git에 대한 간단한 소개와 기본적인 명령어에 대해 정리했었습니다. 그 글은 Git에 대한 책을 읽고 정리한 것으로, 제 기준에서는 Git을 사용하기에 충분한 내용이었습니다.

하지만 Git을 계속 사용하다 보니 .git 폴더에 대한 궁금증이 생겼고, Git에 대해 더 알고 싶은 호기심이 생겼습니다. 이번 포스트에서는 .git에 대한 내용을 다루고자 합니다. 이 내용은 제가 찾아보고 공부한 것을 정리한 글이기 때문에 정확하지 않을 수도 있습니다. Git 문서를 참고하였습니다.
<br>
<br>
<br>

1. [.git directory](#git-directory)
2. [비교](#비교)
3. [Three states and Three trees](#three-states-and-three-trees)
    * [Three states](#three-states)
    * [Three trees](#three-trees)
4. [Summary](#summary)

<br>
<br>

### **.git directory**
.git 폴더는 local work directory에서 숨김 폴더로 있습니다. 결론부터 말하면 .git 폴더는 version database입니다. version database는 사용자가 지금까지 저장했던 버전(스냅샷)들을 모아놓고 사용자가 원하는 버전을 가지고 오고 싶을 때, version databaes에서 해당 시점(버전)의 파일 모양을 가지고 올 수 있습니다. local 컴퓨터에 version database가 있는 이유는 다음과 같습니다.

초기 vcs(version control system)은 다른 폴더에 복사해 놓는 것이었습니다. 사용하기 쉽고 간단했지만, 어느 폴더에 있는지 잊어버거나 실수로 잘못된 파일에 쓰거나 의도하지 않은 파일을 복사할 우려가 있다는 문제가 있었습니다.

이러한 문제를 해결하기 위해 개발자들은 파일의 변경점들을 저장해 놓는 database를 가지는 vcs를 개발했습니다. 대표적으로 RCS(Revision control system)가 있습니다. 하지만, 이 방법에도 문제가 있었습니다. 그건 다른 시스템에서 개발자들과 협력해야 하는 상황이 있을 수 있다는 것이었습니다. 

그래서 개발자들은 cvcs(centralized version control system)을 개발하였습니다. 이는 모든 버전 정보가 포함된 단일 서버가 있고 사용자들은 중앙 위치의 서버(단일 서버)와 통신해서 원하는 시점의 파일 모양을 가져오는 방식입니다. 대표적으로 'Subversion', 'Perforce'가 있습니다. 이러한 방식은 오랫동안 사용되어 왔습니다. 하지만 중앙 서버에 문제가 발생했을 시, 복구되기 전까지 협업을 할 수가 없고 최악의 경우 디스크가 손상되었을 시, 모든 정보들을 잃을 수 있다는 단점이 있었습니다. 

이러한 문제를 보완하고자 나온 것이 dvcs(distributed version control system)입니다. 이는 중앙 서버가 아닌 사용자들의 local 시스템에도 전체 기록(version database/full history)을 가지도록 하는 방식입니다. 이렇게 되면, 중앙 서버가 다운되거나 손상되어도 클라이언트 저장소 중 하나를 서버에 백업하여 복원할 수 있습니다. 'git'은 대표적인 dvcs 중 하나이고 작업 공간에서 숨긴 파일로 있는 .git 폴더는 version database 또는 full history입니다. 
<br>
<br>
<br>

### **비교**
git은 dvcs입니다. dvcs의 장점은 위에서 말했듯이 안정성이 높을 수 있겠지만, cvcs에 비해 속도도 빠르다는 장점도 있습니다. cvcs는 네트워크 기반으로 작동하기 때문에 대기 시간 오버헤드가 있는 반면, dvcs는 local에 프로젝트 history를 저장하기 때문에 거의 즉각적으로 작동하는 것처럼 보입니다. 

이외에도 git은 데이터를 관리할 때, 스냅샷 기술을 이용하고 증분 스냅샷 방식을 사용하는 것 같습니다.(이 부분은 정확하지 않을 수도 있습니다) git은 커밋하거나 저장할 때마다 새로운 공간에 데이터를 복사하지 않고 스냅샷 포인터를 이용해서 수정되지 않은 데이터에 대해 링크를 겁니다. 만약 수정된 데이터가 있다면, 수정된 데이터만 새로운 스냅샷에 저장합니다.

마지막으로 git은 무결성을 가집니다. 모든 것이 체크섬을 적용한 이후 저장됩니다. git은 체크섬을 위해 SHA-1 해시 방식을 이용합니다. 16진수 문자로 구성된 40자 문자열이며 git의 파일 또는 폴더 구조 내용을 기반으로 계산합니다. 흔히 커밋 로그를 확인할 때 나오는 커밋 해쉬값입니다. 
<br>
<br>
<br>

### **Three states and Three trees**
이 내용은 'Git&GitHub' 글에도 부분적으로 정리되어 있지만, git에서 중요한 부분이라서 한 번더 정리하려고 합니다. 

* #### **Three states**
    데이터는 세 상태 'modefied', 'staged', 'committed'가 있습니다. 'Modefied'은 수정되었지만 아직 데이터베이스에 저장되어 있지 않은 상태입니다. 'staged'은 현재 수정된 파일을 다음 커밋 버전(스냅샷)으로 이동하도록 표시했음을 의미합니다. 'committed'는 데이터가 데이터베이스로 안전하게 이동되었음을 의미합니다. 

* #### **Three trees**
    세 가지 주요 섹션 'working tree', 'staging area', 'git directory'가 있습니다. 작업 트리는 프로젝트의 한 버전에 대한 단일 스냅샷입니다. 이러한 파일은 Git 디렉터리의 압축된 데이터베이스에서 꺼내어 사용하거나 수정할 수 있도록 폴더에 배치됩니다.

    스테이징 영역은 일반적으로 'git directory'에 포함된 파일로, 다음 커밋에 포함될 내용에 대한 정보를 저장합니다. 'index'파일에 저장됩니다.

    git directory는 git이 프로젝트의 메타데이터와 개체 데이터베이스를 저장하는 곳입니다. 
<br>
<br>
<br>

### **Summary**
'.git 폴더가 무엇일까'에 대한 궁금증으로 시작한 것에 대한 대답은 다음과 같습니다. .git 폴더가 있는 공간은 프로젝트의 한 버전에 대한 모습(스냅샷)입니다. .git 폴더는 프로젝트의 메타데이터와 개체 데이터 베이스를 저장하고 있습니다. .git 폴더 안에 index(staging area) 파일에는 staged data에 대한 정보가 담겨져 있으며, objects 폴더에는 개체 데이터가 저장되어 있습니다. 또한, 버전의 조타수 역할을 하는 HEAD에 대한 정보도 저장되어 있습니다.