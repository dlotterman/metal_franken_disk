
git clone https://github.com/dlotterman/metal_franken_disk

cd metal_franken_disk

python3 -m venv .venv

source .venv/bin/activate

pip install --upgrade pip

pip install -r requirements.txt
#python3 -m pip install ansible

pre-commit install

pip install -r https://raw.githubusercontent.com/equinix-labs/ansible-collection-equinix/main/requirements.txt

cp equinix.yaml /dev/shm/equinix.yaml
chmod 0700 /dev/shm/equinix.yaml



echo "  - $YOUR_PROJECT_ID_HERE" >> /dev/shm/equinix.yaml

export METAL_AUTH_TOKEN=
