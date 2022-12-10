pipeline {
    // groovy Script で使用する環境パスを定義します
    environment {
        // 構成タスクにある構成パラメータを環境変数に転換します
        IMAGE = sh(returnStdout: true, script: 'echo registry.$image_region.aliyuncs.com/$image_namespace/$image_reponame:$image_tag').trim()
        BRANCH = sh(returnStdout: true, script: 'echo $branch').trim()
    }

    // 今回は土のラベルの構成環境を使用するか定義します。本デモは “slave-pipeline”
    agent {
        node {
            label 'slave-pipeline'
        }
    }

    // "stages"プロジェクト構成の多数のモジュールを定義します。多数の “stage”を追加可能です。多数の “stage”は行列または並列で実行できます
    stages {
        // 一つ目のstageを定義します。コードクローンタスクを完成します
        stage('Git') {
            steps {
                git branch: '${BRANCH}', credentialsId: '', url: 'https://github.com/nancyli2008/jenkins-demo.git'
            }
        }

        //二つ目のstageを追加します， コードパッケージコマンドを実行します
        stage('Package') {
            steps {
                container("maven") {
                    sh "mvn -B -f `pwd`/pom.xml package -DskipTests=true"
                }
            }
        }


        // 三つ目のstageを追加します， コンテナー作成とコマンドPushするを実行します。environmentに定義しているgroovy環境変数を使用しています
        stage('Image Build And Publish') {
            steps {
                container("kaniko") {
                    sh "/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --destination=${IMAGE} --skip-tls-verify"
                }
            }
        }

        // 四つ目のstageを追加します, アプリケーションを指定のK8sクラスターへデプロイします
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh "sed -i 's#IMAGE#${IMAGE}#g' application-demo.yaml"
                    sh "kubectl apply -f application-demo.yaml"
                }
            }
        }
    }
}
