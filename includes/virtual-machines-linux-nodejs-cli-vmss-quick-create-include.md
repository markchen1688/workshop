## <a name="prerequisites"></a>必要條件

您可以取得[免費試用的 Azure 訂用帳戶](https://azure.microsoft.com/pricing/free-trial/)，然後透過 [Azure CLI 1.0](../articles/cli-install-nodejs.md) [連線到您的 Azure 帳戶](../articles/xplat-cli-connect.md)，如果您尚未這麼做的話。 請確定 Azure CLI 是處於 Resource Manager 模式，如下所示：

```azurecli
azure config mode arm
```

## <a name="create-the-scale-set"></a>建立擴展集

現在，使用 `azure vmss quick-create` 命令建立您的擴展集。 下列範例會建立名為 `myVMSS` 的 Linux 擴展集，其中名為 `myResourceGroup` 的資源群組中會有&5; 個 VM 執行個體：

```azurecli
azure vmss quick-create -n myVMSS -g myResourceGroup -l westus \
    -u ops -p P@ssw0rd! \
    -C 5 -Q Canonical:UbuntuServer:16.04-LTS:latest
```

下列範例會使用相同組態建立 Windows 擴展集：

```azurecli
azure vmss quick-create -n myVMSS -g myResourceGroup -l westus \
    -u ops -p P@ssw0rd! \
    -C 5 -Q MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest
```

如果您想要自訂位置或 image-urn，請查看 `azure location list` 和 `azure vm image {list-publishers|list-offers|list-skus|list|show}` 命令。

部署完擴展集之後，此命令就會返回。 此時，擴展集會具有一個帶有 NAT 規則的負載平衡器，這些規則會將負載平衡器上的連接埠 50,000+i 對應至 VM i 上的連接埠 22。 因此，在我們找出負載平衡器的 FQDN 之後，即可透過 SSH 連接到 VM：

```bash
# (if you decide to run this as a script, please invoke using bash)

# list load balancers in the resource group we created
#
# generic syntax:
# azure network lb list -g RESOURCE-GROUP-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network lb list -g negatvmssrg | grep negatvmssrg)
split_line=( $line )
lb_name=${split_line[1]}

# now that we have the name of the load balancer, we can show the details to find which Public IP (PIP) is 
# associated to it
#
# generic syntax:
# azure network lb show -g RESOURCE-GROUP-NAME -n LOAD-BALANCER-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network lb show -g negatvmssrg -n $lb_name | grep loadBalancerFrontEnd)
split_line=( $line )
pip_name=${split_line[4]}

# now that we have the name of the public IP address, we can show the details to find the FQDN
#
# generic syntax:
# azure network public-ip show -g RESOURCE-GROUP-NAME -n PIP-NAME
#
# example with some quick-and-dirty grep-fu to store the result in a variable:
line=$(azure network public-ip show -g negatvmssrg -n $pip_name | grep FQDN)
split_line=( $line )
FQDN=${split_line[3]}

# now that we have the FQDN, we can use ssh on port 50,000+i to connect to VM i (where i is 0-indexed)
#
# example to connct via ssh into VM "0":
ssh -p 50000 negat@$FQDN
```