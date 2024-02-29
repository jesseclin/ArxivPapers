# ArXiv ペーパー読み取り器

以下はアルゴリズムの公式実装です：

```
YouTube: https://www.youtube.com/@ArxivPapers
TikTok: https://www.tiktok.com/@arxiv_papers
Apple Podcasts: https://podcasts.apple.com/us/podcast/arxiv-papers/id1692476016
Spotify: https://podcasters.spotify.com/pod/show/arxiv-papers
```

この作業の主なアイデアは、ArXiv ペーパーの読み取りを簡略化し効率化することです。
もし視覚的な学習者なら、このコードはペーパーを魅力的なビデオ形式に変換します。
もし急いでいて聞くことが好きなら、このコードは音声を生成します。

## 概要

![Teaser image](./imgs/overview.png)

アルゴリズムの主要なステップは次のとおりです：

1. ArXiv ID を与えてペーパーのソースコードをダウンロードします。

2. `latex2html` または `latexmlc` を使用して LaTeX コードを HTML ページに変換します。

3. HTML ページを解析してテキストと方程式を抽出し、表や図などは無視します。

4. ビデオを作成する場合、pdf ページからテキストへのマップとテキストチャンクからページブロックへのマップも作成します。

5. テキストをセクションに分割し、OpenAI GPT API を介して言い換え、簡略化、説明を行います。

6. GPT が生成したテキストをチャンクに分割し、Google API を使用してテキストをオーディオに変換します。

7. 必要なすべての要素を詰め込んで、さらなるビデオ処理のための zip ファイルを作成します。

8. 以前に計算されたテキストブロックマップを使用して、``ffmpeg`` を使用してビデオを作成します。

**注意 1** このコードは、ペーパーの詳細なバージョンと要約されたバージョンの両方を作成できます。

**注意 2** 長いビデオバージョンには、各セクションの後に要約ブロックも含まれます。

**注意 3** 短いビデオバージョンには、ペーパーの要約を自動生成したスライドが含まれます。

**注意 4** コードは、適切な資格情報を提供すると、生成されたオーディオファイルを Google ドライブにアップロードすることもできます。

## セットアップ
- LaTeXML (https://github.com/brucemiller/LaTeXML)
- latex2html (https://github.com/latex2html/latex2html)
- OpenAI key
- ffmpeg (to make videos)
- gcloud: (https://cloud.google.com/sdk/docs/install)
- setup ADC: (https://cloud.google.com/docs/authentication/provide-credentials-adc#how-to)
- Google Text-to-Speech (https://cloud.google.com/text-to-speech)
- Google Drive setup (optional, follow https://github.com/iterative/PyDrive2)

## Python パッケージ
openai, PyPDF2, spacy, tiktoken, pyperclip, google-cloud-texttospeech, pydrive2, pdflatex

## 実行方法

```.bash
# オーディオの作成、短いバージョンと長いバージョンの両方、ビデオの作成準備を行います

python main.py --verbose --include_summary --create_short --create_video --openai_key <your_key> --paperid <arxiv_paper_id> --l2h
```
デフォルトの LaTeX 変換ツール ``latex2html`` は場合によっては失敗することがあります。その場合は ``--l2h`` を削除して ``latexmlc`` を使用してください。また、デフォルトではコードは参考文献までペーパー全体を処理します。早めに終了したい場合は、``--stop_word "experiments"`` を渡してください（例: Experiments セクションの前に終了）。

### 出力 
```
<arxiv_paper_id>_files/
├── final_audio.mp3
├── final_audio_short.mp3
├── abstract.txt
├── zipfile-<time_stamp>.zip
├── ...
├── extracted_orig_text_clean.txt
├── original_text_split_pages.txt
├── original_text_split_sections.txt
├── ...
├── gpt_text.txt
├── gpt_text_short.txt
├── gpt_verb_steps.txt
├── ...
├── slides
    ├── slide1.pdf
    ├── ...
```
出力ディレクトリには、生成されたオーディオファイル、スライド、抽出された元のテキスト、GPT によって生成された出力が、ページまたはセクションごとに分割されて含まれます。出力にはまた、``zipfile-<time_stamp>.zip`` も含まれており、ビデオ生成のためのデータが含まれています。

```.bash
# ArXiv ペーパーから GPT/オーディオ/ビデオ処理なしで元のテキストのみを抽出します

python main.py --verbose --extract_text_only --paperid <arxiv_paper_id>
```

それでは、ビデオを生成する準備ができました：
```.bash
# 上記の結果を元にビデオを生成するために、指定します

python makevideo.py --paperid <arxiv_paper_id>
```

### 出力 
```
output_<time_stamp>/
├── output.mp4
├── output_short.mp4
├── ...
```
出力ディレクトリには、長いビデオと短いビデオの両方のビデオファイルが含まれています。
