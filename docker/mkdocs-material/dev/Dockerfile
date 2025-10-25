FROM python:3.11-slim

WORKDIR /app

# プラグインをインストール
RUN pip install --no-cache-dir \
    # MkDocs本体
    mkdocs==1.5.3 \
    # Material for MkDocs
    mkdocs-material==9.5.15 \
    # 多言語プラグイン
    mkdocs-static-i18n==1.2.0

EXPOSE 8000

CMD ["mkdocs", "serve", "--dev-addr=0.0.0.0:8000"]
