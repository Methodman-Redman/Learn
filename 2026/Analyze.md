# Mitre=hunter
---
## Ready
```
python3 -m venv venv
. venv/bin/activate
pip install stix2 rich
mkdir apt_search
cd apt_search
wget https://raw.githubusercontent.com/mitre-attack/attack-stix-data/master/enterprise-attack/enterprise-attack.json
```
## execute
- `python3 apt_hunter.py T1133,T1190,T1078,T1059.001,T1106,T1218,T1070,T1112,T1222.001,T1003.003,T1555,T1082,T1083,T1057,T1021.001,T1021.002,T1021.004,T1090.003,T1071.001,T1041,T1485,T1529,T1561.002,T1565 --navigator`

## Script
```
#!/usr/bin/env python3

import json
import argparse
import os
from collections import defaultdict
from rich.console import Console
from rich.table import Table

console = Console()
ATTACK_JSON = "enterprise-attack.json"
YARA_DIR = "/home/kali/NTTPs/protections-artifacts/yara/rules"

# MITRE読み込み

def load_attack():

    with open(
        ATTACK_JSON,
        encoding="utf-8"
    ) as f:

        return json.load(f)["objects"]

# DB作成

def parse_attack(objects):

    groups={}
    software={}
    techniques={}
    relations=[]

    for obj in objects:

        if obj["type"]=="intrusion-set":

            groups[obj["id"]]=obj["name"]

        elif obj["type"]=="malware" or obj["type"]=="tool":

            software[obj["id"]]=obj["name"]

        elif obj["type"]=="attack-pattern":

            for ref in obj.get(
                "external_references",
                []
            ):

                if ref.get(
                    "source_name"
                )=="mitre-attack":

                    techniques[
                        obj["id"]
                    ]=ref["external_id"]

        elif obj["type"]=="relationship":

            if obj["relationship_type"]=="uses":

                relations.append(obj)


    return groups,software,techniques,relations

# APT → Technique

def build_apt_technique(
    groups,
    techniques,
    relations
):

    result=defaultdict(set)

    for r in relations:

        if (
            r["source_ref"] in groups
            and
            r["target_ref"] in techniques
        ):

            result[
                groups[r["source_ref"]]
            ].add(
                techniques[
                    r["target_ref"]
                ]
            )

    return result

# APT → Software

def build_apt_software(
    groups,
    software,
    relations
):

    result=defaultdict(set)

    for r in relations:

        if (
            r["source_ref"] in groups
            and
            r["target_ref"] in software
        ):

            result[
                groups[
                    r["source_ref"]
                ]
            ].add(
                software[
                    r["target_ref"]
                ]
            )

    return result

# YARA検索

def search_yara(
    software_list
):

    found=[]

    if not os.path.exists(
        YARA_DIR
    ):
        return found

    files=os.listdir(
        YARA_DIR
    )

    for sw in software_list:

        keyword=sw.lower().replace(
            " ",
            ""
        )

        for f in files:

            if keyword in f.lower().replace(
                "_",
                ""
            ):

                found.append(
                    {
                        "software":sw,
                        "rule":f
                    }
                )

    return found

# Navigator Layer

def create_layer(
    techniques,
    filename="apt_layer.json"
):

    layer={

        "name":
        "APT Hunter Result",

        "version":
        "4.5",

        "domain":
        "enterprise-attack",

        "techniques":[]
    }

    for t in techniques:

        layer["techniques"].append(

            {
                "techniqueID":t,
                "score":100,
                "comment":
                "Detected Technique"
            }
        )

    with open(
        filename,
        "w",
        encoding="utf-8"
    ) as f:

        json.dump(
            layer,
            f,
            indent=2
        )

def main():

    parser=argparse.ArgumentParser()

    parser.add_argument(
        "techniques"
    )

    parser.add_argument(
        "--navigator",
        action="store_true",
        help="Generate MITRE ATT&CK Navigator layer"
    )

    args=parser.parse_args()

    target=set(
        args.techniques.split(",")
    )

    objects=load_attack()

    groups,software,techniques,relations=\
        parse_attack(objects)

    apt_tech=build_apt_technique(
        groups,
        techniques,
        relations
    )

    apt_sw=build_apt_software(
        groups,
        software,
        relations
    )

    result=[]

    for apt,techs in apt_tech.items():

        match=len(
            target & techs
        )

        score=(
            match /
            len(target)
            *
            100
        )

        result.append(
            (
                apt,
                score
            )
        )

    result.sort(
        key=lambda x:x[1],
        reverse=True
    )

    table=Table()

    table.add_column(
        "APT"
    )

    table.add_column(
        "Score"
    )

    for apt,score in result[:10]:

        table.add_row(
            apt,
            f"{score:.1f}%"
        )

    console.print(table)

    # Top APTのTool表示

    top=result[0][0]

    console.print(
        "\n[bold]Top APT:[/bold]",
        top
    )

    tools=apt_sw[top]

    for t in tools:

        console.print(
            " [+]",
            t
        )

    # YARA検索

    rules=search_yara(
        tools
    )

    console.print(
        "\n[bold]YARA Rules[/bold]"
    )

    for r in rules:

        console.print(
            r["software"],
            "=>",
            r["rule"]
        )

    # Navigator

    if args.navigator:

        filename = (
            top
            .replace(" ", "_")
            .replace("/", "_")
            +
            "_result_layer.json"
        )

        create_layer(
            target,
            filename
        )

        console.print(
            "\nNavigator Layer:",
            filename
        )

if __name__=="__main__":
    main()

```

# NW
---
## PacketCapture
- `tshark -D`
- `chmod 777 ~/learn`
- `sudo tshark -i eth0 -w test.pcap`

## JA3S
- `git clone https://github.com/salesforce/ja3.git`
- `cd python`
- `python3 -m venv venv`
- `. venv/bin/activate`
- `pip install -r requirements.txt`
```
pip install packaging
sed -i 's/from distutils.version import LooseVersion/from packaging.version import Version/' ja3s.py
sed -i 's/LooseVersion/Version/g' ja3s.py
```
- `sudo chown kali:kali ~/learn/test.pcap`
- `python3 ja3s.py ~/learn/test.pcap`

## JA4S
- `git clone https://github.com/FoxIO-LLC/ja4.git`
- `python3 ~/learn/ja4/python/ja4.py ~/learn/test.pcap -J`

## Arkime
## apt 
- `sudo apt install arkime -y`
## Set up
- `sudo arkime-first-setup`
  - eth0
- 1. interface の質問
  - Semicolon ';' seperated list of interfaces to monitor<span style="background:rgba(163, 67, 31, 0.2)"> [eth1]</span>
  - 今表示されているインターフェース: br-4c643cf37c17;docker0;eth0;lo。
    - たぶん eth0 だと思うので、こう入力して Enter: eth0
      - Enter（デフォルトのeth1）でも致命的ではありません。後でconfig.iniの interface= を空にしたり、サービスを止めれば大丈夫です。
- 2. Elasticsearch をローカルに入れるか
  - Install Elasticsearch server locally for demo, must have at least 3G of memory, NOT recommended for production use (yes or no) [no]
  - まだElasticsearch/OpenSearchを自分で入れていないなら、ここは:<span style="background:rgba(163, 67, 31, 0.2)">yes</span>
- KaliのVMにメモリ3GB以上割り当ててあるかだけ気にしてください。
- 「Arkime - Configured - Now continue with step 4 in /opt/arkime/README.txt」のようなメッセージが出るはずです。
- OpenSearch/Elasticsearch user [empty is no user] <span style="background:rgba(163, 67, 31, 0.2)">kali</span>
  - 「ArkimeがOpenSearch/Elasticsearchに接続するときのユーザー名」です。
- 何も入れなければ「認証なしで接続を試す」、kali と入れればユーザー名kaliを使います。
  - 今回のセットアップスクリプトは、ローカル用にOpenSearchを一緒に入れて、そこで使うユーザー名・パスワードをここで決めさせようとしている状態だと考えて大丈夫です。
    - 手っ取り早くいくなら、そのまま kali のままで Enter でOKです（後で変えることもできます）。
    - 「OpenSearchのユーザー kali / パスワード kali」という組み合わせを作るイメージです。
- Password to encrypt S2S and other things, don't use spaces [must create one]
  - これはOpenSearchとは別で、「Arkime内部でS2S通信やトークンの暗号化などに使う共通鍵」です。
  - 空欄にはできないので、自分で新しいパスワード文字列を1つ決めて入力します（英数字記号OK・スペースはダメ）。
    - 例: ArkimePcap2026! みたいなもの。
    - 2回同じものを聞かれるのは「確認入力」です。1回目とまったく同じ文字列をもう一度入力してください。

```
sudo wget https://www.wireshark.org/download/automated/data/manuf -O /usr/lib/arkime/etc/oui.txt
sudo chown root:root /usr/lib/arkime/etc/oui.txt
sudo chmod 644 /usr/lib/arkime/etc/oui.txt
sudo /usr/lib/arkime/bin/capture -c /usr/lib/arkime/etc/config.ini -r ~/learn/test.pcap
```

```
sudo systemctl stop arkimeviewer.service
sudo systemctl stop elasticsearch.service
```

## SocketSpy
- `https://github.com/DaiconHacker/SocketSpy`

# File Ana
---
## capa
- `wget https://github.com/mandiant/capa/releases/download/v9.4.0/capa-v9.4.0-linux.zip`
- execute
  - `./capa -v ~/Stowaway/linux_x64_agent`
## strace
- install
  - `sudo apt install strace`
- execute
  - Stop NW interface
  - `strace -f -e trace=network ./DISASTROUS_QUANTITY`

## clamav
- `sudo apt install clamav -y`
  - `sudo clamscan <path to scan directory or file>`

## detect-it-easy
- `sudo apt install detect-it-easy -y`
- `diec -h `

## yara
- `sudo apt install yara -y`
- `git clone https://github.com/elastic/protections-artifacts.git`
## Unpacker
```
git clone https://github.com/anpa1200/Unpacker.git
cd Unpacker
python3 -m venv venv
. venv/bin/activate
pip install -e .
```
- `python scripts/run_unpacker.py /path/to/sample.exe -o ./unpacked`
## unipacker
- `sudo apt install pyenv`
- `pyenv install --list | grep " 3.12"`
- `pyenv install 3.12.10`
- vi ~/.zshrc
```
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```
- `pyenv local 3.12.10`
- `python --version`
- `python3 -m venv venv`
- `. venv/bin/activate`
- `pip uninstall setuptools -y`
- `pip install setuptools==69.5.1`
- `unipacker`

## Unpack
```
 UPX (basic support)
 ASPack
 FSG
 Themida (basic support)
 WinUpack
 Petite
 PESpin
 Armadillo
 PECompact
 NSPack
 MPRESS
```
- `https://github.com/orcastor/unpack`
- `https://github.com/anpa1200/String-Analyzer`
- `https://github.com/M3rcuryLake/Nyxelf`






