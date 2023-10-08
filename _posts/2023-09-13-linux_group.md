---
layout: post
title:  "Linux Group"
tags: "linux"
date:   2023-09-13 00:00:00
---

리눅스 운영체제에서는 Group이라는 기능이 있습니다. 이는 효율성과 보안성을 높여 줍니다. 그룹은 의미 그대로 여러 사용자들을 하나의 그룹으로 묶는 것을 말합니다. 마치 조별 프로젝트를 하는 것과 같이 각각의 조는 본인들의 과제를 진행하고, 서로의 과제에 대해서는 볼 수 없게 하는 것처럼 사용됩니다. (저도 공부 중에 있어서 실무에서 어떻게 사용되는지는 자세하게 모릅니다.) 

<br>
<br>


#### **/etc/group**

현재 생성된 그룹은 /etc/group파일에 그룹이름:x:GID 형식으로 저장되어 있습니다. 사용자 추가 시 그룹 설정을 하지 않으면 GID가 100인 users그룹에 속하게 됩니다. **하지만 레드햇 계열에서는 사용자 계정이 생성될 때 함께 사용자 이름(username)의 그룹이 생성됩니다.** 그리고 각 그룹은 기본적으로 오름차순으로 정렬되어 있습니다.

<br>

#### **그룹 생성 삭제 조회**
그룹을 생성, 삭제, 조회하는 명령어로는 groupadd, groupdel, groups가 있습니다.

<br>

#### **그룹에 사용자 추가**
그룹에 사용자를 추가하는 방법은 3가지가 있습니다. 첫 번째로는 gpassswd 명령어를 사용하는 것이고, 두 번째는 usermod를 사용해 사용자의 프로필을 변경하는 것입니다. 마지막으로, 세 번째는 /ect/group 파일을 수정하는 것입니다. 세 방법은 조금씩 차이가 있습니다.3개 이상의 그룹에 사용자를 추가하고 싶다면, /etc/group 파일을 직접 수정해야 합니다. 그리고 Primary group을 바꾸거나 secondary 그룹을 추가하고 싶을 때는 usermod -g or -G 옵션을 사용하면 됩니다. 여기서 주 그룹은 식별자 그룹을 말합니다. 사용자의 GID는 식별자 그룹의 GID를 의미합니다. 

<br>

#### **wheel(GID=10)**
추가로 wheel(GID=10) 이라는 그룹이 있습니다. wheel 그룹은 관리자 권한을 대행하는 그룹입니다. 일반적으로, 일반 계정은 su나 sudo 명령어는 사용하지 못합니다. 하지만 wheel 그룹에 추가된 사용자는 su와 sudo 명령어를 사용할 수 있습니다. wheel이라는 용어는 큰 힘이나 영향력을 가진 사람을 가리키는 속어인 큰 바퀴에서 파생되었습니다.(해당 내용은 위키피디아 wheel(computing)에서 가져왔습니다)