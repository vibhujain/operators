# Upgrading the KUDO Kafka Operator



KUDO Kafka upgrades work by linking the Kafka cluster `Instance` object to the correct `operatorVersion` object.

We can have multiple operator versions in the same Kubernetes cluster. 
One instance object represents a Kafka Cluster and multiple instances can use the same or different operator versions.

![operator-upgrade-1](./resources/images/operator-upgrade-1.png)

Upgrading can be done by patching the Kafka Instance that holds all the configuration of a specific Kafka cluster and pointing it to a new OperatorVersion object.



![operator-upgrade-1](./resources/images/operator-upgrade-2.png)

The following upgrade procedure assumes one running Kafka cluster and one Kafka operator version already installed (version `kafka-0.1.1`):

Install a new operator version:

```
kubectl kudo install kafka --version=0.2.0 --skip-instance

operator.kudo.k8s.io/kafka unchanged
operatorversion.kudo.k8s.io/v1alpha1/kafka-0.2.0 created
```
Now there are two operator versions installed:
```
kubectl  get operatorversions.kudo.k8s.io

NAME              AGE
kafka-0.1.1       2d2h
kafka-0.2.0       2m6s
```

Check for running instances:

```
kubectl get instances
NAME           AGE
kafka-fc6vzn   29h
```

We can check the plan status of our Kafka cluster `kafka-fc6vzn` with:
```
kubectl kudo plan status --instance=kafka-fc6vzn
Plan(s) for "kafka-fc6vzn" in namespace "default":
.
└── kafka-fc6vzn (Operator-Version: "kafka-0.1.1" Active-Plan: "kafka-fc6vzn-deploy-414458000")
    └── Plan deploy (serial strategy) [COMPLETE]
        └── Phase deploy-kafka (serial strategy) [COMPLETE]
            └── Step deploy (COMPLETE)
```
**Note:** the operator version is `kafka-0.1.1`

To update the Kafka cluster from version `0.1.1` to `0.2.0`:

```
> kubectl patch instance kafka  -p '{"spec":{"operatorVersion":{"name":"kafka-0.2.0"}}}' --type=merge
instance.kudo.k8s.io/kafka patched
```

Check the plan status again:

```
kubectl kudo plan status --instance=kafka
Plan(s) for "kafka" in namespace "default":
.
└── kafka (Operator-Version: "kafka-0.2.0" Active-Plan: "kafka-deploy-575561000")
    └── Plan deploy (serial strategy) [COMPLETE]
        └── Phase deploy-kafka (serial strategy) [COMPLETE]
            └── Step deploy (COMPLETE)
```

**Note:** the operator-version is now `kafka-0.2.0` meaning the Kafka Operator has successfully been upgraded.
