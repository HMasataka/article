# はじめに

※ 読み飛ばして大丈夫です

禁煙に失敗しました。というのも、以前に[Vim やめます](https://qiita.com/Sylba2050/items/af81342dd591b10ec314)という記事を書いたのですが、早々にVim(正確にはneovim)に戻ってきてしまいました。
そちらの記事からはそれなりに時間があいているので割と離れていたかのように思えますが、記事が思いのほかバズってしまい、すぐに戻ってきましたと言いづらかっただけで実際にはVimから完全に離れていたのは数週間程度だったと思います。
今ではVimとVSCodeを8:2くらいの割合で利用しています。(これでも以前と比べVSCodeの割合はかなり増えました)

ありがたいことにただのポエム記事を30000人近くの方に見て頂くことが出来[^1]、twitter、はてブ、vim-jp等々で様々なコメントを頂きました。その中で「おう！また明日な！」「禁煙みたいな感じでしょ？」」「Vimの幻影を求める間はまたVimに戻ってくる」[^2]といった言及が多く、
案外自分以上に自分のことを理解している人が多かったのだなといまさらながら痛感しています。特に「禁煙」という表現がとてもしっくり来たので冒頭にて利用させて頂きました。

さて、これ以上続けるとまたもやポエム記事と化してしまいそうなのでそろそろ本題に入ろうと思います。
なお、今回の記事の内容はおそらくほとんどがvimでも可能だとは思いますが、未検証のためタイトルはneovimとしています。

## 自動で環境構築をする流れ

1. neovimをインストールする(残念ながら手動で行う必要があります)
1. dein.vimを自動でインストールする
1. dein.vimで各種プラグインを自動でインストールする
1. dein.vimで自動インストールしたcoc.nvimで各種Language Serverを自動インストールする
1. 快適なVimライフを送る

なお、neovim のインストールはMacであれば下記で行えます。

```bash
brew install neovim
```

## Vim関連ファイルの構成

僕はホームディレクトリにVim関連のファイルを以下のように配置しています。これ以降の説明はこの配置を前提としたものが含まれるため、必要に応じて適宜読みかえてくだい。

```text
.vim
├── coc-settings.json
├── colors
│   └── 自作カラースキーマ置き場(省略)
├── dein.toml
├── dein_lazy.toml
├── init.vim
├── settings
│   └── 各種プラグインの設定ファイル(省略)
└── tmp
    └── .gitkeep
```

`coc-setting.json`、`init.vim`は`$HOME/.config/nvim/`以下にシンボリックリンクを設定して利用しています。

## dein.vim の自動インストール

vimのプラグイン管理には[dein.vim](https://github.com/Shougo/dein.vim)を利用しています。こちらに関してはドキュメントも充実しており、解説記事が多々存在しているので詳しい解説は省略させていただきます。

dein.vimの自動インストールは`init.vim`にスクリプトを書いて実現しています。
今回はそれに関連する部分のみ抜き出して解説しようと思います。

```vimscript:init.vim
let s:dein_dir = expand('~/.cache/dein')
let s:dein_repo_dir = s:dein_dir . '/repos/github.com/Shougo/dein.vim'
let g:rc_dir = expand('~/.vim')
```

```vimscript:init.vim
if !isdirectory(s:dein_repo_dir)
  execute '!git clone https://github.com/Shougo/dein.vim' s:dein_repo_dir
endif
execute 'set runtimepath^=' . fnamemodify(s:dein_repo_dir, ':p')
```

```vimscript:init.vim
if dein#load_state(s:dein_dir)
  call dein#begin(s:dein_dir)

  " load the file which contain the plugin list
  let s:toml      = g:rc_dir . '/dein.toml'
  let s:lazy_toml = g:rc_dir . '/dein_lazy.toml'

  call dein#load_toml(s:toml, {'lazy': 0})
  call dein#load_toml(s:lazy_toml, {'lazy': 1})

  call dein#end()
  call dein#save_state()
endif
```

```vimscript:init.vim
" automatically install any plug-ins that need to be installed.
if dein#check_install()
  call dein#install()
endif
```

## dein.vim によるプラグインの自動インストール

## coc.nvim によるLanguage Serverの自動インストール

### 自動フォーマット

余談ですが`coc-setting.json`に下記のように記載しておけばファイル保存時に自動でフォーマットしてくれるので便利です。
フォーマットしたいファイルタイプだけを列挙することも可能です。

```json:coc-setting.json
{
    "coc.preferences.formatOnSaveFiletypes": ["*"]
}
```

[^1]: QiitaのView数を参考にしています。実際にはユニークアクセス数では無いと思うのでもっと少ないと思います。
[^2]: 前回の記事中でVSCodeはVimモードで利用していると言及したためだと思われます。現在はVSCode利用時はVimモードは利用していません。
