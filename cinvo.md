#!/usr/bin/env python3.12
"""
listar_render.py - Renderiza um CSV de negocio agrupado por template.

Uso:
    python3.12 listar_render.py <arquivo.csv> [--filtro TEXTO] [--negocio]
"""
import csv
import sys
import argparse
from collections import OrderedDict


def print_table(headers, data_rows):
    widths = [len(str(h)) for h in headers]
    for row in data_rows:
        for i, val in enumerate(row):
            widths[i] = max(widths[i], len(str(val)))

    def fmt_row(vals):
        return "| " + " | ".join(str(v).ljust(widths[i]) for i, v in enumerate(vals)) + " |"

    sep = "| " + " | ".join("-" * widths[i] for i in range(len(headers))) + " |"
    print(fmt_row(headers))
    print(sep)
    for row in data_rows:
        print(fmt_row(row))


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("arquivo")
    parser.add_argument("--filtro", default=None)
    parser.add_argument("--negocio", action="store_true",
                         help="CSV tem coluna 'negocio' extra (modo all)")
    args = parser.parse_args()

    with open(args.arquivo, newline="", encoding="utf-8") as f:
        rows = list(csv.DictReader(f))

    if args.filtro:
        alvo = args.filtro.lower()
        rows = [r for r in rows if alvo in r.get("grupo_proprietario", "").lower()]

    if not rows:
        print("Nenhum resultado encontrado.")
        return

    YELLOW = "\033[1;33m"
    CYAN = "\033[0;36m"
    BOLD = "\033[1m"
    RESET = "\033[0m"

    def key(r):
        if args.negocio:
            return (r.get("negocio", ""), r["grupo_proprietario"], r["template"])
        return (r["grupo_proprietario"], r["template"])

    grupos = OrderedDict()
    for r in rows:
        grupos.setdefault(key(r), []).append(r)

    for k, grupo_rows in grupos.items():
        primeira = grupo_rows[0]
        if args.negocio:
            negocio, grupo, template = k
            titulo = f"{negocio} · {grupo} → {template}"
        else:
            grupo, template = k
            titulo = f"{grupo} → {template}"

        print(f"\n{BOLD}{YELLOW}▸ {titulo}{RESET}")
        print_table(
            ["id_grupo", "id_template"],
            [[primeira.get("id_grupo", ""), primeira.get("id_template", "")]],
        )

        print(f"\n  {CYAN}Etapas:{RESET}")
        etapa_rows = sorted(grupo_rows, key=lambda r: int(r["ordem"]))
        data = [[r["ordem"], r["nome_etapa"], r["id_etapa"]] for r in etapa_rows]
        # indenta a tabela de etapas visualmente
        import io
        buf = io.StringIO()
        old_stdout = sys.stdout
        sys.stdout = buf
        print_table(["ordem", "nome_etapa", "id_etapa"], data)
        sys.stdout = old_stdout
        for linha in buf.getvalue().splitlines():
            print("  " + linha)


if __name__ == "__main__":
    main()
