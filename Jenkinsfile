podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:jdk8
        command:
        - sleep
        args:
        - 99d
      restartPolicy: Never
''') {
      node(POD_LABEL) {
            stage('k8s') {
              git 'https://github.com/SaintamenEdwards/week8.git'
              container('gradle') {
                stage('start calculator') {
                    sh '''
                    pwd
                    set +e
                    ls /bin/test
                    echo "getting kubectl"
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    echo "chmod"
                    chmod +x ./kubectl
                    echo "apply"
                    ./kubectl apply -f calculator.yaml
                    ./kubectl apply -f hazelcast.yaml
                    '''
                }
                stage('test calculator') {
                    sh '''
                    CALCIP=`hostname -i`
                    export CALCIP
                    echo $CALCIP
                    chmod +x gradlew
                    ./gradlew acceptanceTest -D calculator.url=http://calculator-service:8080
                    sh 'test $(curl calculator-service.staging.svc.cluster.local:8080/sum?a=1\\&b=2) -eq 3'
                    sh 'curl calculator-service.staging.svc.cluster.local:8080/div?a=1\\&b=1'
                    '''
                }
              }
            }

      }
    }
