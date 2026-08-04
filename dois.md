#!/bin/bash
#
# listar.sh - Lista arquivos CSV de negócio (agro.csv, seguros.csv, etc) de forma organizada
#
# Uso:
#   ./listar.sh agro                     -> lista dados/agro.csv
#   ./listar.sh agro -f "Agro"           -> filtra por texto na coluna grupo_proprietario
#   ./listar.sh agro --full              -> mostra todas as colunas (incluindo IDs)
#   ./listar.sh agro --cols ordem,nome_etapa
#   ./listar.sh all                      -> junta TODOS os .csv de dados/ numa tabela só
#   ./listar.sh all -f "Consignado"      -> filtra em todos os arquivos
#   ./listar.sh listar                   -> lista quais negócios (arquivos) existem
#
# Requisitos: csvkit (pip install --user csvkit)

set -e

DATA_DIR="./dados"

GREEN='\033[0;32m'
CYAN='\033[0;36m'
YELLOW='\033[1;33m'
RESET='\033[0m'

if [ -z "$1" ]; then
    echo -e "${YELLOW}Uso:${RESET} $0 <negocio|all|listar> [opções]"
    echo ""
    echo "Opções:"
    echo "  -f, --filter <texto>   Filtra linhas onde grupo_proprietario contém <texto>"
    echo "  --full                 Mostra todas as colunas (incluindo IDs)"
    echo "  --cols <col1,col2>     Mostra apenas as colunas informadas"
    echo ""
    echo "Exemplos:"
    echo "  $0 agro"
    echo "  $0 seguros -f Seguros"
    echo "  $0 all"
    echo "  $0 listar"
    exit 1
fi

NOME="$1"
shift

COLS="ordem,nome_etapa,grupo_proprietario,template"
FILTRO=""
MOSTRAR_TUDO=0

while [ "$1" != "" ]; do
    case "$1" in
        -f|--filter)
            FILTRO="$2"
            shift 2
            ;;
        --full)
            MOSTRAR_TUDO=1
            shift
            ;;
        --cols)
            COLS="$2"
            shift 2
            ;;
        *)
            echo "Opção desconhecida: $1"
            exit 1
            ;;
    esac
done

# --- Modo: listar quais negócios existem ---
if [ "$NOME" == "listar" ]; then
    echo -e "${CYAN}Negócios disponíveis em ${DATA_DIR}:${RESET}"
    for f in "$DATA_DIR"/*.csv; do
        [ -e "$f" ] || continue
        base=$(basename "$f" .csv)
        n=$(($(wc -l < "$f") - 1))
        echo -e "  ${GREEN}${base}${RESET}  (${n} etapas)"
    done
    exit 0
fi

# --- Modo: juntar todos os arquivos ---
if [ "$NOME" == "all" ]; then
    TMP=$(mktemp)
    PRIMEIRO=1
    for f in "$DATA_DIR"/*.csv; do
        [ -e "$f" ] || continue
        negocio=$(basename "$f" .csv)
        if [ "$PRIMEIRO" -eq 1 ]; then
            header=$(head -n1 "$f")
            echo "negocio,${header}" > "$TMP"
            PRIMEIRO=0
        fi
        tail -n +2 "$f" | sed "s/^/${negocio},/" >> "$TMP"
    done
    ARQUIVO="$TMP"
    COLS="negocio,${COLS}"
else
    if [ -f "${DATA_DIR}/${NOME}.csv" ]; then
        ARQUIVO="${DATA_DIR}/${NOME}.csv"
    elif [ -f "${NOME}.csv" ]; then
        ARQUIVO="${NOME}.csv"
    else
        echo "Arquivo não encontrado: ${NOME}.csv (procurei em ${DATA_DIR}/ e na pasta atual)"
        echo "Use '$0 listar' para ver os negócios disponíveis."
        exit 1
    fi
fi

echo -e "${CYAN}Fonte:${RESET} ${NOME}"
[ -n "$FILTRO" ] && echo -e "${CYAN}Filtro:${RESET} grupo_proprietario contém \"$FILTRO\""
echo ""

if [ "$MOSTRAR_TUDO" -eq 1 ]; then
    CMD="cat \"$ARQUIVO\""
else
    CMD="csvcut -c $COLS \"$ARQUIVO\""
fi

if [ -n "$FILTRO" ]; then
    CMD="$CMD | csvgrep -c grupo_proprietario -m \"$FILTRO\""
fi

eval "$CMD" | csvlook | (command -v less >/dev/null && less -RS || cat)

[ "$NOME" == "all" ] && rm -f "$TMP"
