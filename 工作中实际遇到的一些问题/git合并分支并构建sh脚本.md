```shell
#!/bin/bash

jenkins_name=$jenkins_name
jenkins_token=$jenkins_token
function check_services_to_build() {
  if [[ -z "${jenkins_name}" || -z "${jenkins_token}" ]]; then
    echo "请检查环境变量 jenkins_name、jenkins_token，设置之后才可以build服务"
    return
  fi
  echo "将使用${jenkins_name}:${jenkins_token}进行构建"
  # 获取自上次提交以来修改的文件列表
  local changed_files=$(git diff --name-only HEAD~1 HEAD)

  # 初始化需要构建的服务列表
  local services_to_build=()

  # 分析修改的文件，确定涉及的模块
  for file in $changed_files; do
    if [[ $file == admin/* ]]; then
      services_to_build+=("admin")
    fi

  done

  # 去除重复的服务名，并添加前缀和后缀
  local unique_services_to_build=()
  for service in $(echo "${services_to_build[@]}" | tr ' ' '\n' | sort -u); do
    unique_services_to_build+=("hp-${service}-api")
  done

  for service in ${unique_services_to_build[*]}; do
    echo "需要构建的服务：$service"
    curl -sS -X POST "http://you-jenkins-url/job/$service/build" --user $jenkins_name:$jenkins_token
  done

}

# 定义源分支和目标分支
source_branch=feature/v1.1
target_branch=test

# 获取最新的远程分支
git fetch origin $source_branch

# 记录当前的分支名，以便之后返回
current_branch=$(git rev-parse --abbrev-ref HEAD)

# 创建临时分支，基于目标分支
git checkout -b temp origin/$target_branch
git pull origin/$target_branch

# 将源分支合并到临时分支
git merge origin/$source_branch

# 检查是否有冲突
conflicts=$(git diff --name-only --diff-filter=U)
if [ -n "$conflicts" ]; then
  echo "存在合并冲突，需要手动解决："
  echo $conflicts
  # 如果存在冲突，回到之前的分支并退出脚本
  git checkout $current_branch
  exit 1
fi

# 如果没有冲突，将临时分支推送到目标分支
if git push origin temp:$target_branch; then
    echo "Push successful"
else
    echo "Push failed"
    exit 1
fi

# 构建修改了的服务
check_services_to_build

# 回到源分支
git checkout $current_branch

# 删除临时分支
git branch -d temp

echo "合并并推送成功。"

```

在实际开发过程中，因为是多人小组在一个dev分支上开发，需要构建的时候merger到test分支上构建，所以步骤如下： 
1. dev -> test
2. 切换到jenkens页面，选择对应的服务执行构建
3. 构建dev分支

现在有了脚本后，步骤如下：
1. 在dev分支上执行脚本，脚本会从orgin/dev 分支拉一个临时分支，然后合并到test分支，然后根据提交文件，进行筛选对应的构建服务，构建test分支，在切换回dev分支，删除临时分支

### 教程
1. you-jenkins-url 替换成你实际的jenkins地址，使用f12抓包即可
2. source_branch=feature/v1.1 "feature/v1.1"替换成你实际的源分支
3. 设置环境变量
jenkins_name 你的jenkins用户名
jenkins_token  你的jenkins token，这个是需要jenkins生成的
4. 如果是在windows上，需要安装git bash，然后在git bash中执行脚本即可