"""Discordへ「今日のアナログホラーアイデア」を投稿するスクリプト。

ローカル実行:
    python main.py

投稿せず内容だけ確認:
    python main.py --preview
"""

from __future__ import annotations

import argparse
import logging
import os
import sys
from datetime import date, datetime
from zoneinfo import ZoneInfo

import requests
from dotenv import load_dotenv

from prompts import SYSTEM_PROMPT, build_user_prompt, fallback_idea


JST = ZoneInfo("Asia/Tokyo")
DEFAULT_MODEL = "gpt-5-mini"
DISCORD_EMBED_DESCRIPTION_LIMIT = 4096


def setup_logging() -> None:
    """GitHub Actionsでも読みやすい形式でログを出します。"""

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s [%(levelname)s] %(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
    )


def load_settings() -> tuple[str | None, str | None, str]:
    """環境変数を読み込みます。

    ローカルでは .env があれば自動で読みます。
    GitHub Actionsでは Secrets から環境変数として渡されます。
    """

    load_dotenv()
    webhook_url = os.getenv("DISCORD_WEBHOOK_URL")
    api_key = os.getenv("OPENAI_API_KEY")
    model = os.getenv("OPENAI_MODEL") or DEFAULT_MODEL
    return webhook_url, api_key, model


def generate_idea(api_key: str | None, model: str, today: date) -> str:
    """OpenAI APIで企画案を生成します。

    APIキーがない場合や、API呼び出しで失敗した場合はフォールバック案を返します。
    これにより「Discord投稿の動作確認」だけならAPIキーなしでもできます。
    """

    if not api_key:
        logging.warning("OPENAI_API_KEY が未設定です。フォールバック案を使用します。")
        return fallback_idea(today)

    try:
        from openai import OpenAI

        client = OpenAI(api_key=api_key)
        response = client.responses.create(
            model=model,
            instructions=SYSTEM_PROMPT,
            input=build_user_prompt(today),
            max_output_tokens=1800,
        )
        idea = response.output_text.strip()
    except Exception:
        logging.exception("OpenAI APIでの生成に失敗しました。フォールバック案を使用します。")
        return fallback_idea(today)

    if not idea:
        logging.warning("OpenAI APIの返答が空でした。フォールバック案を使用します。")
        return fallback_idea(today)

    return idea


def build_discord_payload(message: str) -> dict[str, object]:
    """Discord Webhookへ送るJSONを作ります。"""

    lines = message.strip().splitlines()
    title = lines[0].strip() if lines else "【今日のアナログホラー案】"
    description = "\n".join(lines[1:]).strip() if len(lines) > 1 else message.strip()

    if len(description) > DISCORD_EMBED_DESCRIPTION_LIMIT:
        logging.warning(
            "Discordの埋め込み本文の上限に近いため、%s文字に短縮します。",
            DISCORD_EMBED_DESCRIPTION_LIMIT,
        )
        description = (
            description[: DISCORD_EMBED_DESCRIPTION_LIMIT - 20].rstrip()
            + "\n\n（文字数調整済み）"
        )

    return {
        "embeds": [
            {
                "title": title[:256],
                "description": description,
                "color": 0x2F3136,
            }
        ]
    }


def post_to_discord(webhook_url: str | None, message: str) -> None:
    """Discord Webhookへ投稿します。"""

    if not webhook_url:
        raise RuntimeError(
            "DISCORD_WEBHOOK_URL が未設定です。READMEの手順に沿って "
            ".env または GitHub Secrets に登録してください。"
        )

    response = requests.post(
        webhook_url,
        json=build_discord_payload(message),
        timeout=20,
    )

    if response.status_code >= 400:
        raise RuntimeError(
            "Discordへの投稿に失敗しました。"
            f"status={response.status_code}, body={response.text}"
        )


def parse_args() -> argparse.Namespace:
    """コマンドライン引数を読みます。"""

    parser = argparse.ArgumentParser(
        description="今日のアナログホラーアイデアをDiscordへ投稿します。"
    )
    parser.add_argument(
        "--preview",
        action="store_true",
        help="Discordへ投稿せず、生成された内容だけ表示します。",
    )
    return parser.parse_args()


def main() -> int:
    setup_logging()
    args = parse_args()
    webhook_url, api_key, model = load_settings()
    today = datetime.now(JST).date()

    logging.info("今日の日付: %s JST", today.isoformat())
    logging.info("使用モデル: %s", model if api_key else "fallback")

    idea = generate_idea(api_key=api_key, model=model, today=today)

    if args.preview:
        print(idea)
        return 0

    try:
        post_to_discord(webhook_url=webhook_url, message=idea)
    except Exception:
        logging.exception("投稿処理でエラーが発生しました。")
        return 1

    logging.info("Discordへの投稿が完了しました。")
    return 0


if __name__ == "__main__":
    sys.exit(main())
