Lancer la commande `ls` pour affichers les fichiers

> Pensez à sauvegarder !

![Tux](https://i.pinimg.com/736x/1a/12/fe/1a12febff254956a51efc9379c7afe64--gandalf-linux.jpg)

```bash
#!/bin/bash
set -e
PLUGIN_NAME="openscap"
PRETTY_NAME="OpenSCAP Policies"
CONFIGURATION_PATH=/opt/rudder/share/plugins/$PLUGIN_NAME

# Don't do anything during an upgrade
RUDDER_PLUGIN_UPGRADE="$1"

if [ "$RUDDER_PLUGIN_UPGRADE" == "upgrade" ]
then
  echo "Skipping postinstall during upgrade of OpenSCAP plugin, as Technique already exists"
  exit 0
fi

# Code below should be mostly common between the plugins
SOURCE_DIR=${CONFIGURATION_PATH}/techniques
CONFIG_REPO=/var/rudder/configuration-repository

CATEGORY="$PRETTY_NAME plugin"
C_CATEGORY=$(echo $CATEGORY | sed "s/[^a-zA-Z0-9_]/_/g")
FOLDERS="techniques/$C_CATEGORY"

mkdir -p $CONFIG_REPO/techniques/$C_CATEGORY

cat <<EOT > $CONFIG_REPO/techniques/$C_CATEGORY/category.xml
<xml>
  <name>$CATEGORY</name>
  <description>
    Techniques from the $CATEGORY
  </description>
</xml>
EOT

# Import Generic Methods
cd $CONFIG_REPO
git reset
for file in $FOLDERS
do
  git add $file
done
git commit -m "$CATEGORY installation"
/opt/rudder/bin/rudder-fix-repository-permissions

# Make extra scripts executables
chmod +x $CONFIGURATION_PATH/remove_configuration

# Import Techniques
curl --silent -k --header "Content-type: application/json"   --header "X-API-Token: $(cat /var/rudder/run/api-token)" --request PUT https://localhost/rudder/api/latest/techniques --data "@${CONFIGURATION_PATH}/techniques/plugin_openscap_report.json"
 ```
