---
title: "Pythonプロジェクトのディレクトリ構造設計：個人開発からexe配布まで"
emoji: "📁"
type: "tech"
topics: ["python", "pyinstaller", "projectstructure", "exe", "個人開発"]
published: false
---

# はじめに

個人開発でPythonプロジェクトを進める際、適切なディレクトリ構造は開発効率と保守性を大きく左右します。特に、複数の実行可能プログラムを含み、最終的にexe形式で配布することを前提とした場合、計画的な構造設計が不可欠です。

本記事では、開発からテスト、そしてexe化によるリリースまでをスムーズに行うための実践的なディレクトリ構造を提案します。

# 推奨ディレクトリ構造

## 基本構造

```
my_project/
├── src/                    # ソースコード
│   ├── apps/              # 各実行可能プログラム
│   │   ├── app1/
│   │   │   ├── __init__.py
│   │   │   └── main.py
│   │   ├── app2/
│   │   │   ├── __init__.py
│   │   │   └── main.py
│   │   └── launcher/      # ランチャーアプリ
│   │       ├── __init__.py
│   │       └── main.py
│   ├── common/            # 共通モジュール
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── utils.py
│   │   └── logger.py
│   └── resources/         # リソースファイル
│       ├── images/
│       ├── data/
│       └── config/
├── tests/                 # テストコード
│   ├── unit/             # 単体テスト
│   ├── integration/      # 統合テスト
│   └── conftest.py
├── build/                # ビルド関連
│   ├── specs/           # PyInstallerスペックファイル
│   └── scripts/         # ビルドスクリプト
├── dist/                # 配布用成果物
├── docs/                # ドキュメント
├── requirements/        # 依存関係
│   ├── base.txt        # 基本依存
│   ├── dev.txt         # 開発用依存
│   └── build.txt       # ビルド用依存
├── .gitignore
├── README.md
├── setup.py
└── pyproject.toml
```

## 各ディレクトリの役割

### src/apps/ - アプリケーションモジュール
個別の実行可能プログラムを格納します。各アプリは独立したディレクトリとして管理し、`main.py`をエントリーポイントとします。

```python
# src/apps/app1/main.py
import sys
from pathlib import Path

# プロジェクトルートをPythonパスに追加
sys.path.insert(0, str(Path(__file__).parent.parent.parent))

from common import config, logger

def main():
    log = logger.get_logger(__name__)
    log.info("App1 started")
    # アプリケーションロジック
    
if __name__ == "__main__":
    main()
```

### src/common/ - 共通モジュール
複数のアプリケーションで共有するモジュールを配置します。設定管理、ログ出力、ユーティリティ関数などが含まれます。

```python
# src/common/config.py
import json
from pathlib import Path

class Config:
    def __init__(self):
        self.base_dir = Path(__file__).parent.parent
        self.config_path = self.base_dir / "resources" / "config" / "settings.json"
        self._load_config()
    
    def _load_config(self):
        if self.config_path.exists():
            with open(self.config_path, 'r', encoding='utf-8') as f:
                self.data = json.load(f)
        else:
            self.data = {}
```

### src/launcher/ - ランチャーアプリケーション
複数のアプリケーションを統合的に管理するランチャーを実装します。

```python
# src/apps/launcher/main.py
import tkinter as tk
from tkinter import ttk
import subprocess
import sys
from pathlib import Path

class LauncherApp:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("プロジェクトランチャー")
        
        # アプリケーション定義
        self.apps = {
            "アプリ1": "app1/main.py",
            "アプリ2": "app2/main.py",
        }
        
        self.setup_ui()
    
    def setup_ui(self):
        for name, path in self.apps.items():
            btn = ttk.Button(
                self.root,
                text=name,
                command=lambda p=path: self.launch_app(p)
            )
            btn.pack(pady=5, padx=20)
    
    def launch_app(self, app_path):
        base_dir = Path(__file__).parent.parent
        script_path = base_dir / app_path
        subprocess.Popen([sys.executable, str(script_path)])
    
    def run(self):
        self.root.mainloop()

if __name__ == "__main__":
    app = LauncherApp()
    app.run()
```

# ビルド設定

## PyInstallerスペックファイル

各アプリケーション用のスペックファイルを`build/specs/`に配置します。

```python
# build/specs/launcher.spec
# -*- mode: python ; coding: utf-8 -*-

block_cipher = None

a = Analysis(
    ['../../src/apps/launcher/main.py'],
    pathex=['../../src'],
    binaries=[],
    datas=[
        ('../../src/resources', 'resources'),
    ],
    hiddenimports=['common'],
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)

pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.zipfiles,
    a.datas,
    [],
    name='ProjectLauncher',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=False,
    icon='../../src/resources/images/icon.ico'
)
```

## ビルドスクリプト

```python
# build/scripts/build_all.py
import subprocess
import shutil
from pathlib import Path

def build_app(spec_file, output_name):
    """個別アプリケーションのビルド"""
    print(f"Building {output_name}...")
    
    subprocess.run([
        "pyinstaller",
        "--clean",
        "--noconfirm",
        str(spec_file)
    ], check=True)
    
    # 成果物を移動
    dist_dir = Path("dist")
    build_output = dist_dir / output_name
    
    if build_output.exists():
        shutil.rmtree(build_output)
    
    shutil.move(
        f"dist/{output_name}.exe",
        dist_dir / f"{output_name}.exe"
    )

def main():
    """全アプリケーションのビルド"""
    specs_dir = Path(__file__).parent.parent / "specs"
    
    # 各アプリケーションをビルド
    apps = [
        ("launcher.spec", "ProjectLauncher"),
        ("app1.spec", "App1"),
        ("app2.spec", "App2"),
    ]
    
    for spec_file, output_name in apps:
        build_app(specs_dir / spec_file, output_name)
    
    print("All builds completed!")

if __name__ == "__main__":
    main()
```

# 開発ワークフロー

## 1. 環境セットアップ

```bash
# 仮想環境の作成
python -m venv venv

# 仮想環境の有効化 (Windows)
venv\Scripts\activate

# 依存関係のインストール
pip install -r requirements/dev.txt
```

## 2. 開発時の実行

```bash
# 個別アプリの実行
python src/apps/app1/main.py

# ランチャーの実行
python src/apps/launcher/main.py

# テストの実行
pytest tests/
```

## 3. ビルドとリリース

```bash
# ビルド用依存関係のインストール
pip install -r requirements/build.txt

# 全アプリケーションのビルド
python build/scripts/build_all.py

# 配布用ZIPの作成
python build/scripts/create_release.py
```

# ベストプラクティス

## 1. パス管理
exe化後もリソースファイルに正しくアクセスできるよう、パス管理を適切に行います。

```python
import sys
from pathlib import Path

def get_resource_path(relative_path):
    """リソースファイルの絶対パスを取得"""
    if hasattr(sys, '_MEIPASS'):
        # PyInstallerでexe化された場合
        return Path(sys._MEIPASS) / relative_path
    else:
        # 開発環境の場合
        return Path(__file__).parent.parent / relative_path
```

## 2. 設定ファイルの外部化
exe化後も設定を変更できるよう、設定ファイルは外部に配置します。

```python
def get_config_path():
    """設定ファイルのパスを取得"""
    if getattr(sys, 'frozen', False):
        # exe実行時は同じディレクトリ
        return Path(sys.executable).parent / "config.json"
    else:
        # 開発時はリソースディレクトリ
        return Path(__file__).parent / "resources/config/config.json"
```

## 3. ログ出力
デバッグやトラブルシューティングのため、適切なログ出力を実装します。

```python
# src/common/logger.py
import logging
from pathlib import Path
import sys

def get_logger(name):
    logger = logging.getLogger(name)
    
    if not logger.handlers:
        # ログディレクトリの作成
        if getattr(sys, 'frozen', False):
            log_dir = Path(sys.executable).parent / "logs"
        else:
            log_dir = Path(__file__).parent.parent.parent / "logs"
        
        log_dir.mkdir(exist_ok=True)
        
        # ハンドラーの設定
        handler = logging.FileHandler(
            log_dir / f"{name}.log",
            encoding='utf-8'
        )
        handler.setFormatter(
            logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        )
        
        logger.addHandler(handler)
        logger.setLevel(logging.INFO)
    
    return logger
```

# まとめ

本記事で提案したディレクトリ構造により、以下のメリットが得られます：

1. **明確な責任分離**：各アプリケーション、共通モジュール、リソースが明確に分離
2. **スケーラビリティ**：新しいアプリケーションの追加が容易
3. **保守性**：統一された構造により、メンテナンスが容易
4. **ビルドの自動化**：スクリプトによる一括ビルドが可能
5. **配布の簡素化**：ランチャーによる統合的な配布

個人開発においても、このような構造化されたアプローチを採用することで、プロジェクトの成長に伴う複雑性を管理し、効率的な開発を継続できます。

プロジェクトの規模や要件に応じて、この構造をカスタマイズして活用してください。