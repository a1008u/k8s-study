# ------------------------------------------------
# 6.7.1 Headless Serviceの作成
# ------------------------------------------------
作成条件(StatefulSetで作成されたPob名でディスカバりする場合)
- 条件１：Serviceのspec.typeがClusterIPであること
- 条件２：Serviceのspec.clusterIPがNoneであること
- 条件３：Serviceのmetadata.nameがStatefulSetのspec.serviceNameと同じであること


# ------------------------------------------------
# 6.7.2 Headless ServiceによるPod名の名前解決
# ------------------------------------------------
あとで

