---
title: "bigtop mpack에 Hive 추가"
author: Kang HyunGu
date: 2022-01-24 19:26:00 +0900
categories: [Bigdata, Hadoop]
tags: [Bigdata]
toc: true
pin: false
comments: true
---

## 1. Stack에 HIVE 추가.
Stack 디렉토리에 HDFS에 metainfo 파일을 보니 common-service라는 디렉토리에 있는 파일을 가지고 설치를 진행하는듯하다.
metainfo.xml 파일
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hdfs_metainfo.png" width="400" height="500"></p>
위의 xml 파일을 보니 common-service가 뭘까 찾다 ambari-server디렉토리에서 아래와 같이 발견했다.
경로 : /var/lib/ambari-server/resources/common-services

<p align="left"> <img src="{{site.url}}/img/posts/bigtop/common_service.png" width="600" height="350"></p>

그래서 /var/lib/ambari-server/resources/mpacks/bgtp-ambari-mpack-1.0.0.0-SNAPSHOT/stacks/BGTP/1.0/services/HIVE 경로에 HIVE 디렉토리를 만들고 아래와 같이 metainfo.xml을 작성해보았다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_metainfo.png" width="600" height="300"></p>

아래 경로에 링크를 걸어줘야하는거 같아 링크를 걸어주었다.
/var/lib/ambari-server/resources/stacks/BGTP/1.0/services
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/stack1.png" width="400" height="100"></p>

- Hive 추가 확인.
아래 처럼 추가가된걸 확인했는데 설치 진행시 아래와 같은 에러가 발생.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/ambari4.png" width="700" height="300"></p>

```
stderr:
Traceback (most recent call last):
  File "/var/lib/ambari-agent/cache/common-services/HIVE/0.12.0.2.0/package/scripts/hcat_client.py", line 79, in <module>
    HCatClient().execute()
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/script.py", line 352, in execute
    method(env)
  File "/var/lib/ambari-agent/cache/common-services/HIVE/0.12.0.2.0/package/scripts/hcat_client.py", line 34, in install
    import params
  File "/var/lib/ambari-agent/cache/common-services/HIVE/0.12.0.2.0/package/scripts/params.py", line 27, in <module>
    from params_linux import *
  File "/var/lib/ambari-agent/cache/common-services/HIVE/0.12.0.2.0/package/scripts/params_linux.py", line 737, in <module>
    enable_ranger_hive = config['configurations']['hive-env']['hive_security_authorization'].lower() == 'ranger'
  File "/usr/lib/ambari-agent/lib/resource_management/libraries/script/config_dictionary.py", line 73, in __getattr__
    raise Fail("Configuration parameter '" + self.name + "' was not found in configurations dictionary!")
resource_management.core.exceptions.Fail: Configuration parameter 'hive_security_authorization' was not found in configurations dictionary!
 stdout:
2022-01-21 17:48:40,826 - Stack Feature Version Info: Cluster Stack=1.0, Command Stack=None, Command Version=None -> 1.0
2022-01-21 17:48:40,827 - Group['hdfs'] {}
2022-01-21 17:48:40,828 - Group['hadoop'] {}
2022-01-21 17:48:40,829 - Group['users'] {}
2022-01-21 17:48:40,829 - User['hive'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,830 - User['zookeeper'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,830 - User['ambari-qa'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,831 - User['hdfs'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hdfs', 'hadoop'], 'uid': None}
2022-01-21 17:48:40,831 - User['yarn'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,832 - User['mapred'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,832 - User['hcat'] {'gid': 'hadoop', 'fetch_nonlocal_groups': True, 'groups': ['hadoop'], 'uid': None}
2022-01-21 17:48:40,833 - File['/var/lib/ambari-agent/tmp/changeUid.sh'] {'content': StaticFile('changeToSecureUid.sh'), 'mode': 0555}
2022-01-21 17:48:40,834 - Execute['/var/lib/ambari-agent/tmp/changeUid.sh ambari-qa /tmp/hadoop-ambari-qa,/tmp/hsperfdata_ambari-qa,/home/ambari-qa,/tmp/ambari-qa,/tmp/sqoop-ambari-qa 0'] {'not_if': '(test $(id -u ambari-qa) -gt 1000) || (false)'}
2022-01-21 17:48:40,840 - Skipping Execute['/var/lib/ambari-agent/tmp/changeUid.sh ambari-qa /tmp/hadoop-ambari-qa,/tmp/hsperfdata_ambari-qa,/home/ambari-qa,/tmp/ambari-qa,/tmp/sqoop-ambari-qa 0'] due to not_if
2022-01-21 17:48:40,841 - Group['hdfs'] {}
2022-01-21 17:48:40,841 - User['hdfs'] {'fetch_nonlocal_groups': True, 'groups': ['hdfs', 'hadoop', u'hdfs']}
2022-01-21 17:48:40,841 - FS Type: HDFS
2022-01-21 17:48:40,842 - Directory['/etc/hadoop'] {'mode': 0755}
2022-01-21 17:48:40,853 - File['/etc/hadoop/conf/hadoop-env.sh'] {'content': InlineTemplate(...), 'owner': 'hdfs', 'group': 'hadoop'}
2022-01-21 17:48:40,854 - Writing File['/etc/hadoop/conf/hadoop-env.sh'] because contents don't match
2022-01-21 17:48:40,854 - Changing owner for /etc/hadoop/conf/hadoop-env.sh from 0 to hdfs
2022-01-21 17:48:40,854 - Changing group for /etc/hadoop/conf/hadoop-env.sh from 0 to hadoop
2022-01-21 17:48:40,854 - Directory['/var/lib/ambari-agent/tmp/hadoop_java_io_tmpdir'] {'owner': 'hdfs', 'group': 'hadoop', 'mode': 01777}
2022-01-21 17:48:40,872 - Repository['BGTP-1.0-repo-2'] {'base_url': 'http://repos.bigtop.apache.org/releases/1.5.0/centos/7/$basearch', 'action': ['prepare'], 'components': [u'BGTP', 'main'], 'repo_template': '[{{repo_id}}]\nname={{repo_id}}\n{% if mirror_list %}mirrorlist={{mirror_list}}{% else %}baseurl={{base_url}}{% endif %}\n\npath=/\nenabled=1\ngpgcheck=0', 'repo_file_name': 'ambari-bgtp-2', 'mirror_list': None}
2022-01-21 17:48:40,878 - Repository[None] {'action': ['create']}
2022-01-21 17:48:40,879 - File['/tmp/tmpwdGb0M'] {'content': '[BGTP-1.0-repo-2]\nname=BGTP-1.0-repo-2\nbaseurl=http://repos.bigtop.apache.org/releases/1.5.0/centos/7/$basearch\n\npath=/\nenabled=1\ngpgcheck=0'}
2022-01-21 17:48:40,879 - Writing File['/tmp/tmpwdGb0M'] because contents don't match
2022-01-21 17:48:40,880 - Rewriting /etc/yum.repos.d/ambari-bgtp-2.repo since it has changed.
2022-01-21 17:48:40,880 - File['/etc/yum.repos.d/ambari-bgtp-2.repo'] {'content': StaticFile('/tmp/tmpwdGb0M')}
2022-01-21 17:48:40,880 - Writing File['/etc/yum.repos.d/ambari-bgtp-2.repo'] because it doesn't exist
2022-01-21 17:48:40,881 - Package['unzip'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2022-01-21 17:48:41,663 - Skipping installation of existing package unzip
2022-01-21 17:48:41,663 - Package['curl'] {'retry_on_repo_unavailability': False, 'retry_count': 5}
2022-01-21 17:48:42,367 - Skipping installation of existing package curl
2022-01-21 17:48:42,623 - Skipping get_stack_version since /usr/bin/distro-select is not yet available
2022-01-21 17:48:42,623 - Stack Feature Version Info: Cluster Stack=1.0, Command Stack=None, Command Version=None -> 1.0
Command failed after 1 tries

```

## 2. hive_security_authorization 에러 해결
에러를 보니 hive_security_authorization 파라미터가 설정이 안되어있어 발생하는거 같아 /var/lib/ambari-server/resources/common-services/HIVE/0.12.0.2.0/configuration/hive-env.xml에 아래와 같이 추가를 해주었다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_security_authorization_add.PNG" width="700" height="300"></p>
진행 하다 보니 hcatalog 서비스 설치시 hcat이라는 유저를 만들기를 시도하는데 에러가 발생해 아래와같이 hive유저로 변경을 해주었다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_env.PNG" width="700" height="300"></p>
## 3. linux_param 파일 수정.
bigtop 유틸로 하이브를 설치하기 때문에 몇가지 디렉토리 경로를 아래와 같이 바꿔준다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/linuxt_param.PNG" width="700" height="300"></p>

## 4. metainfo.xml 수정.
이번엔 hcatalog라는 패키지가 bigtop 레파지토리에 없어서 에러가 발생했다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/error2.PNG" width="700" height="300"></p>
설치하는 패키지가 기본이 아닌 bigtop 레파지토리에서 받기 때문에 metainfo.xml을 아래와 같이 수정을 한다.
webhcat 실행시에도 에러가 발생하여 설치에서 제외시켰다(사용하지 않음)
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/metainfo.PNG" width="500" height="200"></p>

## 5. 설치 성공
ambari에 추가된 모습.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/success.PNG" width="700" height="250"></p>
처음에 hive설치시 에러가 발생해서 확인해보니 /etc/hive/conf파일에 hive-site.xml이 수정이 안되어있어서 접근이 안되는거 같아 수정을 한뒤 아래와 같이 test_db를 만들어 확인해보았다.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive.PNG" width="700" height="250"></p>

## 6. 정리
이번 기회에 ambari stack에 대해 조금 알게되었지만 소스 레벨을 모두 파악한것은 아니라 에러 해결에만 급급하게 설치했던거 같다.
아래 ambari stack관련 메뉴얼과 기존에 있던 hdp 소스를 분석하면 custom stack을 만들어 입맛에 맞게 구성할 수 있을거 같아 분석을 해보는게 좋을것 같다.
아래 내가 적용했던 소스를 github에 올려두었다.


[github링크](https://github.com/snowturtle93/bgtp-mpack)
