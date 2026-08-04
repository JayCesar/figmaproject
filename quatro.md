#!/bin/bash
#
# listar.sh - Lista CSVs de negócio (agro.csv, seguros.csv, etc), agrupando
# automaticamente por template dentro do mesmo grupo_proprietario.
#
# Uso direto:
#   ./listar.sh agro                -> mostra os templates/etapas do agro
#   ./listar.sh agro -f "Agro"      -> filtra por texto no grupo_proprietario
#   ./listar.sh all                 -> junta todos os negócios
#   ./listar.sh listar              -> lista negócios disponíveis
#
# Uso interativo (sem argumentos):
#   ./listar.sh
#
# Requisitos: python3.12 (já disponível), listar_render.py na mesma pasta

set -e

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
RENDER="$SCRIPT_DIR/listar_render.py"
DATA_DIR="./dados"

GREEN='\033[0;32m'
CYAN='\033[0;36m'
YELLOW='\033[1;33m'
RESET='\033[0m'

listar_negocios() {
    echo -e "${CYAN}Negócios disponíveis em ${DATA_DIR}:${RESET}"
    local i=1
    ARQS=()
    for f in "$DATA_DIR"/*.csv; do
        [ -e "$f" ] || continue
        base=$(basename "$f" .csv)
        n=$(($(wc -l < "$f") - 1))
        echo -e "  ${GREEN}${i})${RESET} ${base}  (${n} linhas)"
        ARQS+=("$base")
        i=$((i+1))
    done
}

processar() {
    local nome="$1"
    local filtro="$2"
    local arquivo
    local flag_negocio=""

    if [ "$nome" == "all" ]; then
        TMP=$(mktemp)
        primeiro=1
        for f in "$DATA_DIR"/*.csv; do
            [ -e "$f" ] || continue
            negocio=$(basename "$f" .csv)
            if [ "$primeiro" -eq 1 ]; then
                header=$(head -n1 "$f")
                echo "negocio,${header}" > "$TMP"
                primeiro=0
            fi
            tail -n +2 "$f" | sed "s/^/${negocio},/" >> "$TMP"
        done
        arquivo="$TMP"
        flag_negocio="--negocio"
    else
        if [ -f "${DATA_DIR}/${nome}.csv" ]; then
            arquivo="${DATA_DIR}/${nome}.csv"
        elif [ -f "${nome}.csv" ]; then
            arquivo="${nome}.csv"
        else
            echo "Arquivo não encontrado: ${nome}.csv"
            echo "Use '$0 listar' para ver os negócios disponíveis."
            exit 1
        fi
    fi

    echo -e "${CYAN}Fonte:${RESET} ${nome}"
    [ -n "$filtro" ] && echo -e "${CYAN}Filtro:${RESET} grupo_proprietario contém \"$filtro\""

    if [ -n "$filtro" ]; then
        python3.12 "$RENDER" "$arquivo" --filtro "$filtro" $flag_negocio \
            | (command -v less >/dev/null && less -RS || cat)
    else
        python3.12 "$RENDER" "$arquivo" $flag_negocio \
            | (command -v less >/dev/null && less -RS || cat)
    fi

    [ "$nome" == "all" ] && rm -f "$TMP"
}

# ------------------------------------------------------------------
# Modo interativo (sem argumentos)
# ------------------------------------------------------------------
if [ -z "$1" ]; then
    listar_negocios
    echo ""
    echo -e "${CYAN}Digite o número, o nome do negócio, ou 'all' para ver todos:${RESET}"
    read -r ESCOLHA

    if [[ "$ESCOLHA" =~ ^[0-9]+$ ]]; then
        idx=$((ESCOLHA-1))
        NOME="${ARQS[$idx]}"
    else
        NOME="$ESCOLHA"
    fi

    echo -e "${CYAN}Filtrar por texto no grupo_proprietario? (Enter pra pular):${RESET}"
    read -r FILTRO

    processar "$NOME" "$FILTRO"
    exit 0
fi

# ------------------------------------------------------------------
# Modo direto (com argumentos)
# ------------------------------------------------------------------
NOME="$1"
shift

if [ "$NOME" == "listar" ]; then
    listar_negocios
    exit 0
fi

FILTRO=""
while [ "$1" != "" ]; do
    case "$1" in
        -f|--filter)
            FILTRO="$2"
            shift 2
            ;;
        *)
            echo "Opção desconhecida: $1"
            exit 1
            ;;
    esac
done

processar "$NOME" "$FILTRO"
