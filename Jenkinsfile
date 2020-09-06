podTemplate(yaml: """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: bazel
    image: gcr.io/cloud-marketplace-containers/google/bazel@sha256:ace9881e6e9c5d48b5fd637321361aeffe54000265894a65f7d818dc1065bd80 # 3.5.0
    command:
    - cat
    tty: true
    volumeMounts:
    - name: dockersock
      mountPath: /var/run/docker.sock
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
"""
) {
  node(POD_LABEL) {
    checkout scm

    container("bazel") {
      dir("my_bazel_tc_project") {
        sh "bazel run //:nodejs_image"
      }
    }
  }
}
