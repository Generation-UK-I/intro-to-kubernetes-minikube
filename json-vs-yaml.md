# JSON and YAML
Both JSON and YAML are both formats for storing and transporting data, they are some occasions where an application may support only one format, but often either may be used, and regardless it's easy to convert between them. 

|JSON|YAML|
|---|---|
|Easier for machines to read|Easier for humans to read|
|Complex syntax|Simple syntax|
|Faster in some use cases|Faster in some use cases|

Really it just boils down to YAML being more human-friendly, and JSON being machine-friendly, but many developers, over time, become very familiar with JSON, and can scan/read it rapidly.

Below is a snippet from an ARM template for deploying a VM in both JSON and YAML:

### JSON
```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2021-03-01",
  "name": "myVM1",
  "location": "[parameters('location')]",
  "dependsOn": ["myNIC1"],
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_B1s"
    },
    "osProfile": {
      "computerName": "myVM1",
      "adminUsername": "[parameters('adminUsername')]",
      "adminPassword": "[parameters('adminPassword')]"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "18.04-LTS",
        "version": "latest"
      },
      "osDisk": {
        "createOption": "FromImage"
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC1')]"
        }
      ]
    }
  }
}
```

### YAML
```yaml
type: Microsoft.Compute/virtualMachines
apiVersion: 2021-03-01
name: myVM1
location: "[parameters('location')]"
dependsOn:
  - myNIC1
properties:
  hardwareProfile:
    vmSize: Standard_B1s
  osProfile:
    computerName: myVM1
    adminUsername: "[parameters('adminUsername')]"
    adminPassword: "[parameters('adminPassword')]"
  storageProfile:
    imageReference:
      publisher: Canonical
      offer: UbuntuServer
      sku: 18.04-LTS
      version: latest
    osDisk:
      createOption: FromImage
  networkProfile:
    networkInterfaces:
      - id: "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC1')]"
```

