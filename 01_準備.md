# kubectlのインストール
自分のmacにkubectlコマンドは入っていたがversion古くなっていそうだったのでもう一度インストールから。

`curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
`  
`chmod +x ./kubectl`  
`sudo mv ./kubectl /usr/local/bin/kubectl`  

# EKSクラスタの作成
次に練習用のクラスタを作成する。
GCPでなく、AWSを選んだ理由は個人的な知識の問題で、AWSの方が知ってることも多く、ハマりが少なそうだったから。
https://docs.aws.amazon.com/ja_jp/eks/latest/userguide/getting-started-console.html
こちらを参考にして作成する。
AWSの場合はコントロールプレーンに対して課金がある。つまりクラスタがあるだけで課金されるので、基本的に学習が終わったからクラスタを削除する方針でいく。
手順
- IAM Roleの作成
- クラスタで利用するVPCの作成（CloudFormationテンプレート利用）
- クラスタ作成

クラスタ作成には少し時間がかかる。

# kubeconfigの作成
awsclieを用いてkubeconfigを作成する。
`aws eks --region ap-northeast-1 update-kubeconfig --name ${cluster_name}`で作成される。
kubeconfigは`${HOME}/.kube/config`にできる。

kubectl get svcで結果が帰ってくればOK。