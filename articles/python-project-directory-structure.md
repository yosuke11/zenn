---
title: "Pythonプロジェクトのディレクトリ構造設計：個人開発からexe配布まで"
emoji: "📁"
type: "tech"
topics: ["python", "pyinstaller", "projectstructure", "exe", "個人開発"]
published: true
---

# はじめに

個人開発でPythonプロジェクトを進める際、適切なディレクトリ構造は開発効率と保守性を大きく左右します。特に、複数の実行可能プログラムを含み、最終的にexe形式で配布することを前提とした場合、計画的な構造設計が不可欠です。

本記事では、開発からテスト、そしてexe化によるリリースまでをスムーズに行うための実践的なディレクトリ構造を提案します。

# 推奨ディレクトリ構造

## 実用的でシンプルな構造

```
my_project/
├── src/
│   ├── apps/              # 各実行可能プログラム
│   │   ├── __init__.py
│   │   ├── app1_data_analyzer.py
│   │   ├── app2_batch_processor.py
│   │   ├── app3_realtime_monitor.py
│   │   ├── app4_report_generator.py
│   │   ├── app5_data_converter.py
│   │   └── launcher.py
│   ├── lib/               # 共通ライブラリ（シンプルに1階層）
│   │   ├── __init__.py
│   │   ├── processing.py  # メイン処理ロジック
│   │   ├── utils.py       # ユーティリティ関数
│   │   ├── config.py      # 設定管理
│   │   └── fast_calc.pyx  # Cython高速化（必要な場合）
│   └── resources/         # リソースファイル
│       ├── config.json
│       └── templates/
├── tests/                 # テストコード
│   ├── __init__.py
│   ├── test_processing.py
│   └── test_utils.py
├── build/                 # ビルド関連
│   ├── build_exe.py      # exe化スクリプト
│   └── app_specs/        # 各アプリのspecファイル
├── dist/                  # 配布用成果物（自動生成）
├── setup.py              # Cythonビルド用
├── requirements.txt      # 依存パッケージ
├── README.md
└── .gitignore
```

## 各ファイルの具体的な実装例

### 1. src/apps/app1_data_analyzer.py - データ分析アプリ
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
データ分析アプリケーション
数値データから画像を生成し、類似度を計算する
"""
import sys
from pathlib import Path

# srcディレクトリをPythonパスに追加（重要！）
sys.path.insert(0, str(Path(__file__).parent.parent))

from lib import processing, utils, config
import tkinter as tk
from tkinter import filedialog, messagebox
import numpy as np

def main():
    """メインエントリーポイント"""
    # 設定読み込み
    cfg = config.load_config()
    
    # GUIアプリケーション
    root = tk.Tk()
    root.title("データ分析ツール")
    root.geometry("600x400")
    
    def analyze_data():
        # ファイル選択
        file_path = filedialog.askopenfilename(
            filetypes=[("CSVファイル", "*.csv"), ("すべて", "*.*")]
        )
        if not file_path:
            return
        
        try:
            # データ読み込みと処理
            data = utils.load_csv_data(file_path)
            processor = processing.ImageProcessor()
            
            # 画像生成
            image = processor.data_to_image(data, dtype="numerical")
            
            # 結果表示
            messagebox.showinfo("完了", f"画像生成完了\nサイズ: {image.shape}")
            
        except Exception as e:
            messagebox.showerror("エラー", str(e))
    
    # UIセットアップ
    tk.Button(root, text="データ分析開始", command=analyze_data, 
              width=20, height=2).pack(pady=20)
    tk.Button(root, text="終了", command=root.quit, 
              width=20, height=2).pack(pady=10)
    
    root.mainloop()

if __name__ == "__main__":
    main()
```

### 2. src/apps/launcher.py - ランチャーアプリケーション
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
プロジェクトランチャー
すべてのアプリケーションを統合管理
"""
import sys
from pathlib import Path
import subprocess
import tkinter as tk
from tkinter import ttk
import json

sys.path.insert(0, str(Path(__file__).parent.parent))

from lib import config

class LauncherApp:
    def __init__(self):
        self.root = tk.Tk()
        self.root.title("プロジェクトランチャー")
        self.root.geometry("400x500")
        
        # アプリケーション定義（同じディレクトリ内のアプリを自動検出）
        self.apps = self._discover_apps()
        self.setup_ui()
    
    def _discover_apps(self):
        """appsディレクトリ内のアプリを自動検出"""
        apps_dir = Path(__file__).parent
        apps = {}
        
        for py_file in apps_dir.glob("app*.py"):
            if py_file.name != "launcher.py":
                # ファイル名から表示名を生成
                display_name = py_file.stem.replace("_", " ").title()
                apps[display_name] = py_file.name
        
        return apps
    
    def setup_ui(self):
        # タイトル
        title_label = ttk.Label(self.root, text="アプリケーション選択", 
                               font=("Arial", 16, "bold"))
        title_label.pack(pady=20)
        
        # アプリケーションボタン
        for name, filename in self.apps.items():
            btn = ttk.Button(
                self.root,
                text=name,
                command=lambda f=filename: self.launch_app(f),
                width=30
            )
            btn.pack(pady=5)
        
        # 終了ボタン
        ttk.Separator(self.root, orient='horizontal').pack(fill='x', pady=20)
        ttk.Button(self.root, text="終了", command=self.root.quit, 
                  width=30).pack(pady=10)
    
    def launch_app(self, filename):
        """アプリケーションを起動"""
        app_path = Path(__file__).parent / filename
        
        # 新しいプロセスで起動
        subprocess.Popen([sys.executable, str(app_path)])
        
        # ログ出力
        print(f"起動: {filename}")
    
    def run(self):
        self.root.mainloop()

if __name__ == "__main__":
    app = LauncherApp()
    app.run()
```

### 3. src/lib/processing.py - メイン処理ロジック
```python
# -*- coding: utf-8 -*-
"""
画像処理と類似度計算のコアロジック
"""
import cv2
import numpy as np
from typing import Union, List, Dict, Tuple
import warnings

# Cythonモジュールのインポート（オプション）
try:
    from . import fast_calc
    HAS_CYTHON = True
except ImportError:
    HAS_CYTHON = False
    warnings.warn("Cythonモジュールが見つかりません。通常速度で動作します。")

class ImageProcessor:
    """画像処理の主要クラス"""
    
    def __init__(self, use_cython: bool = True):
        self.use_cython = use_cython and HAS_CYTHON
        self.kernels = self._init_kernels()
    
    def _init_kernels(self) -> Dict[str, np.ndarray]:
        """各種カーネルの初期化"""
        return {
            'gaussian': cv2.getGaussianKernel(5, 1.0) @ cv2.getGaussianKernel(5, 1.0).T,
            'sobel_x': np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], dtype=np.float32),
            'sobel_y': np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]], dtype=np.float32),
            'laplacian': np.array([[0, 1, 0], [1, -4, 1], [0, 1, 0]], dtype=np.float32)
        }
    
    def data_to_image(self, data: Union[List, np.ndarray], 
                     dtype: str = "numerical", 
                     size: Tuple[int, int] = (128, 128)) -> np.ndarray:
        """データを画像に変換"""
        if dtype == "numerical":
            return self._numerical_to_image(data, size)
        elif dtype == "categorical":
            return self._categorical_to_image(data, size)
        else:
            raise ValueError(f"サポートされていないデータ型: {dtype}")
    
    def _numerical_to_image(self, data: Union[List[float], np.ndarray], 
                           size: Tuple[int, int]) -> np.ndarray:
        """数値データを画像に変換"""
        # numpy配列に変換
        if not isinstance(data, np.ndarray):
            data = np.array(data, dtype=np.float32)
        
        # データを正規化（0-255）
        data_norm = ((data - data.min()) / (data.max() - data.min()) * 255).astype(np.uint8)
        
        # リサイズして画像化
        if self.use_cython:
            # Cython版の高速リシェイプ
            image = fast_calc.fast_reshape_to_image(data_norm, size[0], size[1])
        else:
            # 通常のリシェイプ
            # データが不足する場合は0パディング
            total_pixels = size[0] * size[1]
            if len(data_norm) < total_pixels:
                data_norm = np.pad(data_norm, (0, total_pixels - len(data_norm)))
            elif len(data_norm) > total_pixels:
                data_norm = data_norm[:total_pixels]
            
            image = data_norm.reshape(size)
        
        return image
    
    def _categorical_to_image(self, data: List[str], 
                             size: Tuple[int, int]) -> np.ndarray:
        """カテゴリデータを画像に変換"""
        # カテゴリを数値に変換
        unique_categories = list(set(data))
        category_map = {cat: idx for idx, cat in enumerate(unique_categories)}
        
        # 数値配列に変換
        numerical_data = [category_map[cat] for cat in data]
        
        # 数値データとして画像化
        return self._numerical_to_image(numerical_data, size)
    
    def calculate_similarity(self, img1: np.ndarray, img2: np.ndarray, 
                           method: str = "mse") -> float:
        """2つの画像の類似度を計算"""
        # サイズを合わせる
        if img1.shape != img2.shape:
            img2 = cv2.resize(img2, (img1.shape[1], img1.shape[0]))
        
        # フィルタ適用
        filtered1 = cv2.filter2D(img1, -1, self.kernels['gaussian'])
        filtered2 = cv2.filter2D(img2, -1, self.kernels['gaussian'])
        
        # 類似度計算
        if method == "mse":
            mse = np.mean((filtered1 - filtered2) ** 2)
            similarity = 1 / (1 + mse)
        elif method == "ssim":
            # 構造的類似性指標
            similarity = self._calculate_ssim(filtered1, filtered2)
        else:
            raise ValueError(f"サポートされていない手法: {method}")
        
        return float(similarity)
    
    def _calculate_ssim(self, img1: np.ndarray, img2: np.ndarray) -> float:
        """SSIM（構造的類似性指標）を計算"""
        # 簡易版SSIM実装
        c1 = 0.01 ** 2
        c2 = 0.03 ** 2
        
        mu1 = cv2.GaussianBlur(img1, (11, 11), 1.5)
        mu2 = cv2.GaussianBlur(img2, (11, 11), 1.5)
        
        mu1_sq = mu1 ** 2
        mu2_sq = mu2 ** 2
        mu1_mu2 = mu1 * mu2
        
        sigma1_sq = cv2.GaussianBlur(img1 ** 2, (11, 11), 1.5) - mu1_sq
        sigma2_sq = cv2.GaussianBlur(img2 ** 2, (11, 11), 1.5) - mu2_sq
        sigma12 = cv2.GaussianBlur(img1 * img2, (11, 11), 1.5) - mu1_mu2
        
        ssim = ((2 * mu1_mu2 + c1) * (2 * sigma12 + c2)) / \
               ((mu1_sq + mu2_sq + c1) * (sigma1_sq + sigma2_sq + c2))
        
        return float(np.mean(ssim))
```

### 4. src/lib/utils.py - ユーティリティ関数
```python
# -*- coding: utf-8 -*-
"""
ユーティリティ関数群
"""
import csv
import json
from pathlib import Path
from typing import List, Dict, Any
import numpy as np

def load_csv_data(filepath: str) -> List[float]:
    """CSVファイルからデータを読み込み"""
    data = []
    with open(filepath, 'r', encoding='utf-8') as f:
        reader = csv.reader(f)
        for row in reader:
            # 数値データの列を抽出（最初の列と仮定）
            try:
                value = float(row[0])
                data.append(value)
            except (ValueError, IndexError):
                continue
    
    if not data:
        raise ValueError("有効なデータが見つかりません")
    
    return data

def save_result(result: Dict[str, Any], output_path: str) -> None:
    """結果をJSON形式で保存"""
    output_path = Path(output_path)
    output_path.parent.mkdir(parents=True, exist_ok=True)
    
    with open(output_path, 'w', encoding='utf-8') as f:
        json.dump(result, f, ensure_ascii=False, indent=2)

def get_timestamp() -> str:
    """タイムスタンプを取得"""
    from datetime import datetime
    return datetime.now().strftime("%Y%m%d_%H%M%S")

def ensure_dir(path: str) -> Path:
    """ディレクトリが存在しない場合は作成"""
    path = Path(path)
    path.mkdir(parents=True, exist_ok=True)
    return path
```

### 5. src/lib/config.py - 設定管理
```python
# -*- coding: utf-8 -*-
"""
設定管理モジュール
"""
import json
from pathlib import Path
from typing import Dict, Any

# デフォルト設定
DEFAULT_CONFIG = {
    "image_size": [128, 128],
    "similarity_method": "mse",
    "use_cython": True,
    "output_dir": "output",
    "log_level": "INFO"
}

def get_config_path() -> Path:
    """設定ファイルのパスを取得"""
    # srcの親ディレクトリ（プロジェクトルート）を基準にする
    base_dir = Path(__file__).parent.parent.parent
    return base_dir / "src" / "resources" / "config.json"

def load_config() -> Dict[str, Any]:
    """設定を読み込み"""
    config_path = get_config_path()
    
    if config_path.exists():
        with open(config_path, 'r', encoding='utf-8') as f:
            user_config = json.load(f)
        # デフォルト設定とマージ
        config = DEFAULT_CONFIG.copy()
        config.update(user_config)
        return config
    else:
        # 設定ファイルがない場合はデフォルトを使用
        save_config(DEFAULT_CONFIG)
        return DEFAULT_CONFIG

def save_config(config: Dict[str, Any]) -> None:
    """設定を保存"""
    config_path = get_config_path()
    config_path.parent.mkdir(parents=True, exist_ok=True)
    
    with open(config_path, 'w', encoding='utf-8') as f:
        json.dump(config, f, ensure_ascii=False, indent=2)

def get(key: str, default: Any = None) -> Any:
    """設定値を取得"""
    config = load_config()
    return config.get(key, default)
```

### 6. src/lib/fast_calc.pyx - Cython高速化モジュール（オプション）
```python
# -*- coding: utf-8 -*-
# cython: language_level=3
"""
Cythonによる高速化処理
コンパイルが必要: python setup.py build_ext --inplace
"""
import numpy as np
cimport numpy as np
cimport cython

# NumPy C-APIの初期化
np.import_array()

@cython.boundscheck(False)
@cython.wraparound(False)
def fast_reshape_to_image(np.ndarray[np.uint8_t, ndim=1] data, 
                         int width, int height):
    """高速な1次元配列から2次元画像への変換"""
    cdef int i, j, idx
    cdef int data_len = data.shape[0]
    cdef np.ndarray[np.uint8_t, ndim=2] result = np.zeros((height, width), dtype=np.uint8)
    
    for i in range(height):
        for j in range(width):
            idx = i * width + j
            if idx < data_len:
                result[i, j] = data[idx]
            else:
                result[i, j] = 0  # パディング
    
    return result

@cython.boundscheck(False)
@cython.wraparound(False)
def fast_convolution_2d(np.ndarray[np.float32_t, ndim=2] image,
                       np.ndarray[np.float32_t, ndim=2] kernel):
    """高速2D畳み込み演算"""
    cdef int img_h = image.shape[0]
    cdef int img_w = image.shape[1]
    cdef int ker_h = kernel.shape[0]
    cdef int ker_w = kernel.shape[1]
    cdef int pad_h = ker_h // 2
    cdef int pad_w = ker_w // 2
    
    # 結果配列
    cdef np.ndarray[np.float32_t, ndim=2] result = np.zeros_like(image)
    
    cdef int i, j, m, n
    cdef float sum_val
    
    for i in range(pad_h, img_h - pad_h):
        for j in range(pad_w, img_w - pad_w):
            sum_val = 0.0
            for m in range(ker_h):
                for n in range(ker_w):
                    sum_val += image[i + m - pad_h, j + n - pad_w] * kernel[m, n]
            result[i, j] = sum_val
    
    return result
```

# ビルド設定

## setup.py - Cythonビルド設定
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
Cythonモジュールのビルド設定
使用方法: python setup.py build_ext --inplace
"""
from setuptools import setup, Extension
from Cython.Build import cythonize
import numpy as np
import sys
from pathlib import Path

# プロジェクトのルートディレクトリ
ROOT_DIR = Path(__file__).parent

# Cython拡張モジュールの定義
extensions = [
    Extension(
        name="src.lib.fast_calc",  # モジュールの完全な名前
        sources=["src/lib/fast_calc.pyx"],  # Cythonソースファイル
        include_dirs=[np.get_include()],  # NumPyのヘッダファイルパス
        language="c",  # C言語を使用（C++の場合は"c++"）
    )
]

# セットアップ設定
setup(
    name="MyProjectCython",
    ext_modules=cythonize(
        extensions,
        compiler_directives={
            'language_level': "3",  # Python 3を使用
            'boundscheck': False,   # 境界チェックを無効化（高速化）
            'wraparound': False,    # 負のインデックスを無効化（高速化）
        }
    ),
    zip_safe=False,
)

# ビルド後の説明を表示
if "build_ext" in sys.argv:
    print("\n" + "="*50)
    print("Cythonモジュールのビルドが完了しました！")
    print("生成されたファイル:")
    print("  - src/lib/fast_calc.*.pyd (Windows)")
    print("  - src/lib/fast_calc.*.so (Linux/Mac)")
    print("="*50 + "\n")
```

## requirements.txt - 依存パッケージ
```
# 基本パッケージ
numpy>=1.20.0
opencv-python>=4.5.0

# GUI
tkinter  # 通常はPythonに含まれている

# ビルド用（オプション）
Cython>=0.29.0  # Cython高速化を使用する場合
pyinstaller>=5.0  # exe化用

# テスト用（開発時のみ）
pytest>=7.0.0
pytest-cov>=3.0.0
```

## build/build_exe.py - exe化スクリプト

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
exe化ビルドスクリプト
全アプリケーションを一括でexe化する
"""
import os
import sys
import shutil
import subprocess
from pathlib import Path
import json

# プロジェクトルート
ROOT_DIR = Path(__file__).parent.parent
SRC_DIR = ROOT_DIR / "src"
DIST_DIR = ROOT_DIR / "dist"
BUILD_DIR = ROOT_DIR / "build"

class ExeBuilder:
    def __init__(self):
        self.apps_dir = SRC_DIR / "apps"
        self.specs_dir = BUILD_DIR / "app_specs"
        self.specs_dir.mkdir(exist_ok=True)
    
    def create_spec_file(self, app_name: str, app_path: Path) -> Path:
        """各アプリ用のspecファイルを生成"""
        spec_content = f'''
# -*- mode: python ; coding: utf-8 -*-
# {app_name}のビルド設定

block_cipher = None

a = Analysis(
    ['{app_path.as_posix()}'],
    pathex=['{SRC_DIR.as_posix()}'],
    binaries=[],
    datas=[
        ('{(SRC_DIR / "resources").as_posix()}', 'resources'),
        ('{(SRC_DIR / "lib").as_posix()}', 'lib'),
    ],
    hiddenimports=['lib', 'lib.processing', 'lib.utils', 'lib.config'],
    hookspath=[],
    hooksconfig={{}},
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
    name='{app_name}',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    upx_exclude=[],
    runtime_tmpdir=None,
    console=False,  # GUIアプリの場合False、CLIアプリの場合True
    disable_windowed_traceback=False,
    target_arch=None,
    codesign_identity=None,
    entitlements_file=None,
)
'''
        
        spec_path = self.specs_dir / f"{app_name}.spec"
        spec_path.write_text(spec_content, encoding='utf-8')
        return spec_path
    
    def build_app(self, app_file: Path) -> bool:
        """個別アプリをビルド"""
        app_name = app_file.stem
        print(f"\n{'='*50}")
        print(f"ビルド開始: {app_name}")
        print(f"{'='*50}")
        
        # specファイル作成
        spec_path = self.create_spec_file(app_name, app_file)
        
        # PyInstallerでビルド
        cmd = [
            sys.executable, "-m", "PyInstaller",
            "--clean",
            "--noconfirm",
            "--distpath", str(DIST_DIR),
            "--workpath", str(BUILD_DIR / "work"),
            str(spec_path)
        ]
        
        try:
            result = subprocess.run(cmd, check=True, capture_output=True, text=True)
            print(f"✓ {app_name} のビルドが成功しました")
            return True
        except subprocess.CalledProcessError as e:
            print(f"✗ {app_name} のビルドが失敗しました")
            print(f"エラー: {e.stderr}")
            return False
    
    def build_all(self):
        """全アプリケーションをビルド"""
        print("全アプリケーションのビルドを開始します...")
        
        # distディレクトリをクリーン
        if DIST_DIR.exists():
            shutil.rmtree(DIST_DIR)
        DIST_DIR.mkdir(exist_ok=True)
        
        # アプリケーションファイルを検索
        app_files = list(self.apps_dir.glob("app*.py"))
        app_files.append(self.apps_dir / "launcher.py")
        
        success_count = 0
        for app_file in app_files:
            if self.build_app(app_file):
                success_count += 1
        
        print(f"\n{'='*50}")
        print(f"ビルド完了: {success_count}/{len(app_files)} 成功")
        print(f"成果物: {DIST_DIR}")
        print(f"{'='*50}")
        
        # 配布用ZIPを作成
        if success_count == len(app_files):
            self.create_release_zip()
    
    def create_release_zip(self):
        """配布用ZIPファイルを作成"""
        from datetime import datetime
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        zip_name = f"MyProject_Release_{timestamp}"
        
        print(f"\n配布用ZIPを作成中: {zip_name}.zip")
        shutil.make_archive(
            str(DIST_DIR / zip_name),
            'zip',
            DIST_DIR
        )
        print(f"✓ 配布用ZIPを作成しました: {zip_name}.zip")

if __name__ == "__main__":
    builder = ExeBuilder()
    builder.build_all()
```

## .gitignore - Git除外設定
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
*.pyd
.Python
venv/
env/

# Cython
*.c
*.cpp
build/
*.egg-info/

# PyInstaller
dist/
build/
*.spec

# IDE
.vscode/
.idea/
*.swp
*.swo

# プロジェクト固有
output/
logs/
*.log
.DS_Store
Thumbs.db
```

# 開発ワークフロー

## 1. 環境セットアップ

```bash
# 仮想環境の作成
python -m venv venv

# 仮想環境の有効化 (Windows)
venv\Scripts\activate

# 仮想環境の有効化 (Mac/Linux)
source venv/bin/activate

# 依存関係のインストール
pip install -r requirements.txt

# Cythonモジュールのビルド（オプション）
python setup.py build_ext --inplace
```

## 2. 開発時の実行

```bash
# 個別アプリの実行
python src/apps/app1_data_analyzer.py

# ランチャーから実行
python src/apps/launcher.py

# テストの実行
pytest tests/

# カバレッジ付きテスト
pytest tests/ --cov=src.lib --cov-report=html
```

## 3. exe化と配布

```bash
# 全アプリケーションのexe化
python build/build_exe.py

# 特定のアプリだけexe化する場合
pyinstaller --onefile --windowed src/apps/launcher.py
```

## 4. プロジェクト初期化コマンド

新しいプロジェクトを始める際のコマンド集：

```bash
# プロジェクトディレクトリ構造の作成
mkdir -p src/{apps,lib,resources}
mkdir -p tests build/app_specs
touch src/__init__.py src/apps/__init__.py src/lib/__init__.py
touch README.md requirements.txt setup.py .gitignore

# Gitリポジトリの初期化
git init
git add .
git commit -m "Initial project structure"
```

# ベストプラクティス

## 1. パス管理の統一
開発環境とexe環境の両方で動作するパス解決：

```python
# src/lib/utils.pyに追加
import sys
from pathlib import Path

def get_project_root() -> Path:
    """プロジェクトルートディレクトリを取得"""
    if getattr(sys, 'frozen', False):
        # exe化されている場合
        return Path(sys.executable).parent
    else:
        # 開発環境の場合（srcの親ディレクトリ）
        return Path(__file__).parent.parent.parent

def get_resource_path(relative_path: str) -> Path:
    """リソースファイルの絶対パスを取得"""
    if hasattr(sys, '_MEIPASS'):
        # PyInstallerの一時ディレクトリ
        return Path(sys._MEIPASS) / relative_path
    else:
        return get_project_root() / "src" / relative_path
```

## 2. エラーハンドリング
適切なエラー処理でユーザーフレンドリーに：

```python
# src/apps/app1_data_analyzer.pyの改良版
import sys
import traceback
from tkinter import messagebox

def main():
    try:
        # アプリケーションのメイン処理
        app = DataAnalyzerApp()
        app.run()
    except Exception as e:
        # エラー詳細をログに記録
        error_msg = f"エラーが発生しました:\n{str(e)}\n\n詳細:\n{traceback.format_exc()}"
        
        # ログファイルに保存
        log_path = Path(sys.executable).parent / "error.log" if getattr(sys, 'frozen', False) else "error.log"
        with open(log_path, 'a', encoding='utf-8') as f:
            f.write(f"\n{'='*50}\n{datetime.now()}\n{error_msg}\n")
        
        # ユーザーに通知
        messagebox.showerror("エラー", f"エラーが発生しました。\n詳細は {log_path} を確認してください。")
        sys.exit(1)
```

## 3. メモリ管理
大量データ処理時のメモリ最適化：

```python
# src/lib/processing.pyに追加
import gc

class ImageProcessor:
    def process_large_dataset(self, data_list: List[np.ndarray]) -> List[float]:
        """大量データのバッチ処理"""
        results = []
        batch_size = 100  # メモリに応じて調整
        
        for i in range(0, len(data_list), batch_size):
            batch = data_list[i:i + batch_size]
            
            # バッチ処理
            for data in batch:
                result = self.calculate_similarity(data, self.reference_image)
                results.append(result)
            
            # メモリ解放
            del batch
            gc.collect()
        
        return results
```

## 4. テストの書き方

```python
# tests/test_processing.py
import pytest
import numpy as np
from src.lib.processing import ImageProcessor

class TestImageProcessor:
    @pytest.fixture
    def processor(self):
        return ImageProcessor(use_cython=False)  # テストではCython無効
    
    def test_data_to_image_numerical(self, processor):
        data = [1.0, 2.0, 3.0, 4.0]
        image = processor.data_to_image(data, "numerical", (2, 2))
        
        assert image.shape == (2, 2)
        assert image.dtype == np.uint8
    
    def test_calculate_similarity_identical(self, processor):
        img = np.ones((10, 10), dtype=np.uint8) * 128
        similarity = processor.calculate_similarity(img, img)
        
        assert similarity == 1.0  # 同じ画像なので完全一致
```

# トラブルシューティング

## よくある問題と解決方法

### 1. ModuleNotFoundError: No module named 'lib'
```python
# 原因：Pythonパスが設定されていない
# 解決方法：各アプリの先頭に以下を追加
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent))
```

### 2. exe化後にリソースファイルが見つからない
```python
# specファイルのdatasにリソースを追加
datas=[
    ('src/resources', 'resources'),
    ('src/lib', 'lib'),  # libフォルダも含める
]
```

### 3. OpenCVのImportError
```bash
# opencv-python-headlessを使用（GUI不要の場合）
pip uninstall opencv-python
pip install opencv-python-headless
```

### 4. Cythonモジュールがインポートできない
```python
# オプショナルにしてフォールバックを実装
try:
    from . import fast_calc
    HAS_CYTHON = True
except ImportError:
    HAS_CYTHON = False
    # 通常のPython実装を使用
```

# まとめ

本記事で提案したシンプルなディレクトリ構造により、以下のメリットが得られます：

1. **シンプルさ**：フラットな構造でファイルを見つけやすい
2. **実用性**：5-6個のアプリに最適な規模
3. **拡張性**：必要に応じて構造を複雑化できる
4. **保守性**：共通処理をlibフォルダにまとめて管理
5. **exe化対応**：ビルドスクリプトで簡単に配布可能

個人開発では「シンプルに始めて、必要に応じて成長させる」アプローチが最も効果的です。

この構造をベースに、あなたのプロジェクトに合わせてカスタマイズしてください。