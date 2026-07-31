# inouelab-skills

井上研で共有する、AI コーディングエージェント用のスキル置き場。

**特定のエージェントにも OS にも依存しない。** スキルの中身は素の Markdown で書き、各エージェントへの繋ぎ込みは `install.py` が生成する薄いアダプタに閉じ込めてある。Claude Code / Codex / GitHub Copilot / Cursor / Gemini CLI、あるいはそれ以外でも同じ中身が使える。インストーラは Python 3.8 以降の標準ライブラリだけで動き、macOS / Linux / Windows で同じように使える。

各スキルの内容は、それぞれのディレクトリの `README.md` を見ること。

## スキル一覧

- [slide-review](slide-review/)

## 導入

```sh
git clone git@github.com:inouelab-waseda/inouelab-skills.git
cd inouelab-skills
python3 install.py <provider>
```

Windows（PowerShell / コマンドプロンプト）では `py -3 install.py <provider>`。以降の例の `python3` は読み替えること。

`<provider>` に使っているエージェントを指定する:

| provider | エージェント | 生成先 | 範囲 |
|---|---|---|---|
| `claude` | Claude Code | `~/.claude/skills/<name>/SKILL.md` | ユーザ全体 |
| `codex` | OpenAI Codex CLI | `~/.codex/prompts/<name>.md` | ユーザ全体 |
| `copilot` | GitHub Copilot | `<dest>/.github/prompts/<name>.prompt.md` | リポジトリ単位 |
| `cursor` | Cursor | `<dest>/.cursor/rules/<name>.mdc` | リポジトリ単位 |
| `gemini` | Gemini CLI | `~/.gemini/commands/<name>.toml` | ユーザ全体 |
| `manual` | それ以外 | 標準出力 | — |

```sh
python3 install.py claude                              # 全スキルを導入
python3 install.py claude slide-review                 # 特定のスキルだけ
python3 install.py copilot --dest ~/work/my-project    # リポジトリ単位のものは設置先を指定
python3 install.py cursor --dry-run                    # 何がどこに書かれるか確認するだけ
```

`~` にあたる場所は OS ごとに解決される（Windows なら `C:\Users\<name>`）。生成物に埋め込まれるパスの区切りは、どの OS でも `/` に統一される。

導入後、多くのエージェントでは `/<skill-name>` で呼べる。対応していない場合は「slide-review を使って」と頼めばよい。

`manual` はどのエージェントにも合わせず、登録用のテキストを表示するだけ。上表にないツールを使っている場合は、これを各ツールのカスタム指示欄に貼る。

### 更新

生成されるのは**スキル本体への絶対パスを持つポインタ**なので、スキルの中身の更新は `git pull` するだけで全員に反映される。`install.py` の再実行が要るのは、スキルの `name` / `description` を変えたときと、新しいスキルが増えたときだけ。

### 外す

生成されたファイルを消せばよい（`~/.claude/skills/slide-review/` など。上表の生成先を参照）。リポジトリ側は消えない。

なお `install.py` は、自分が生成したファイル以外は上書きしない。同名のものを自分で作っていた場合は、警告を出してスキップする。

## 構成

1スキル = リポジトリ直下の1ディレクトリ。

```
<skill-name>/
  skill.yaml        # name と description。アダプタ生成に使われる
  instructions.md   # スキル本体。エージェントが読む手順
  README.md         # 人が読む説明。何をするスキルか、どう使うか
  references/       # 詳細な資料。instructions.md から必要なときだけ読まれる
```

`instructions.md` は常にコンテキストに載るので短く保つ。分量のあるものは `references/` に置き、`instructions.md` から「このタイミングで読め」と指示する。

**プロバイダ固有の記述をここに書かないこと。** 特定のツール名、そのツール専用のツール名（`Read` など）、独自のファイル形式に依存すると、他のエージェントで動かなくなる。`references/` からの参照は `instructions.md` からの相対パスで書く。

## スキルを追加する

1. リポジトリ直下に `<name>/` を作る
2. `skill.yaml` に `name` と `description` を書く。`description` には**いつ使うか**を書く。多くのエージェントはこれだけを見て起動を判断するので、想定される呼ばれ方（別名・関連語）を含める。何をするかの説明だけでは起動しない
3. `instructions.md` に手順を書く
4. `README.md` に人向けの説明を書く。この top-level README には一覧のリンクだけ追加する
5. `python3 install.py <provider> --dry-run` で生成物を確認してから導入する
