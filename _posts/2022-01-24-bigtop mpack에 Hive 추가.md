---
title: "bigtop mpack에 Hive 추가"
author: Kang HyunGu
date: 2022-01-24 19:26:00 +0900
categories: [Bigdata, Hadoop]
tags: [Bigdata]
toc: true
pin: true
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
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_metainfo.png" width="900" height="400"></p>

아래 경로에 링크를 걸어줘야하는거 같아 링크를 걸어주었다.
/var/lib/ambari-server/resources/stacks/BGTP/1.0/services
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/stack1.png" width="600" height="200"></p>

- Hive 추가 확인.
아래 처럼 추가가된걸 확인했는데 설치 진행시 아래와 같은 에러가 발생.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/ambari4.png" width="700" height="300"></p>
<pre><code>

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

</code></pre>

2 HIVE 사용하기 위한 common-service 커스텀
2.1
/var/lib/ambari-server/resources/common-services/HIVE/0.12.0.2.0/package/scripts/params_linux.py 파일 수정.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_custom1.png" width="900" height="400"></p>
/var/lib/ambari-server/resources/common-services/HIVE/0.12.0.2.0/metainfo.xml 파일 수정.
hcatalog -> hive-hcatalog.noarch (bigtop 패키지)로 변경.
<p align="left"> <img src="{{site.url}}/img/posts/bigtop/hive_custom2.png" width="900" height="400"></p>
