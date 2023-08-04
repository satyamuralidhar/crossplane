**Install AWS Provider:**

    provider1:
    reference:
    https://marketplace.upbound.io/providers/crossplane-contrib/provider-aws/v0.42.0

    cat <<EOF | kubectl apply -f -
    apiVersion: pkg.crossplane.io/v1
    kind: Provider
    metadata:
    name: contrib-provider-aws
    spec:
    package: xpkg.upbound.io/crossplane-contrib/provider-aws:v0.42.0
    EOF

    provider2:
    cat <<EOF | kubectl apply -f -
    apiVersion: pkg.crossplane.io/v1
    kind: Provider
    metadata:
    name: upbound-provider-aws
    spec:
    package: xpkg.upbound.io/upbound/provider-aws:v0.27.0
    EOF


**Create admin role using AWS IAM:**

    $ touch aws-creds.conf update below configuration

    [default]
    aws_access_key_id = <aws_access_key>
    aws_secret_access_key = <aws_secret_key>

**Create a Secret**

    $ kubectl create secret generic awscreds -n crossplane-system --from-file=creds=./aws-creds.conf

    $ kubectl describe secret awscreds -n crossplane-system


**Initialize AWS Provider:**

    cat <<EOF | kubectl apply -f -
    apiVersion: aws.upbound.io/v1beta1
    kind: ProviderConfig
    metadata:
    name: default
    spec:
    credentials:
        source: Secret
        secretRef:
        namespace: crossplane-system
        name: awscreds
        key: creds
    EOF

    another module: 

    cat <<EOF | kubectl apply -f -
    apiVersion: aws.crossplane.io/v1beta1
    kind: ProviderConfig
    metadata:
    name: awsconfig
    spec:
    credentials:
        source: Secret
        secretRef:
        namespace: crossplane-system
        name: awscreds
        key: creds
    EOF




**Create Managed Resource:**

    Reference: https://docs.crossplane.io/latest/getting-started/provider-aws/

    bucket=$(echo "crossplane-bucket-"$(head -n 4096 /dev/urandom | openssl sha1 | tail -c 10))
    cat <<EOF | kubectl apply -f -
    apiVersion: s3.aws.upbound.io/v1beta1
    kind: Bucket
    metadata:
    name: $bucket
    spec:
    forProvider:
        region: us-east-2
    providerConfigRef:
        name: default
    EOF


**Create a bucket in AWS:**   

    $ kubectl get buckets
    
**Delete Bucket:**

    $ kubectl delete bucket $bucket


