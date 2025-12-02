# =====================================
# Script de configuração - Menu
# =====================================

# --- Função para detectar o grupo Everyone/Todos ---
function Get-GrupoEveryone {
    $gruposTexto = whoami /groups | Out-String

    if ($gruposTexto -match "Todos") {
        return "Todos"
    }
    elseif ($gruposTexto -match "Everyone") {
        return "Everyone"
    }
    else {
        Write-Host "Nenhum grupo 'Todos' ou 'Everyone' localizado. Usando SID S-1-1-0." -ForegroundColor Yellow
        return "S-1-1-0"   # SID universal do Everyone
    }
}

$GrupoEveryone = Get-GrupoEveryone

# --- Função para ativar SMB 1.0 ---
function Ativar-SMB1 {
    Write-Host "Ativando SMB 1.0..." -ForegroundColor Yellow
    Enable-WindowsOptionalFeature -Online -FeatureName "SMB1Protocol" -NoRestart
    Write-Host "SMB 1.0 ativado!" -ForegroundColor Green
}

# --- Função para criar usuário administrador ---
function Criar-UsuarioAdmin {
    param(
        [string]$NomeUsuario = "scanner"
    )

    Write-Host "Criando usuário '$NomeUsuario'..." -ForegroundColor Yellow
    $senha = Read-Host "Digite a senha do usuário $NomeUsuario" -AsSecureString

    if (-not (Get-LocalUser -Name $NomeUsuario -ErrorAction SilentlyContinue)) {
        New-LocalUser -Name $NomeUsuario -Password $senha -FullName $NomeUsuario -Description "Usuário criado pelo script"
        Add-LocalGroupMember -Group "Administrators" -Member $NomeUsuario

        Write-Host "Usuário '$NomeUsuario' criado e adicionado ao grupo Administrators!" -ForegroundColor Green
    } else {
        Write-Host "Usuário '$NomeUsuario' já existe! (não será recriado)" -ForegroundColor Cyan
    }
}

# --- Função para criar pasta compartilhada ---
function Criar-PastaCompartilhada {
    $pasta = "C:\Scanner"
    $share = "Scanner"
    $usuario = "scanner"

    # Criar pasta
    if (-not (Test-Path $pasta)) {
        Write-Host "Criando pasta C:\Scanner..." -ForegroundColor Yellow
        New-Item -Path $pasta -ItemType Directory | Out-Null
    } else {
        Write-Host "Pasta já existe. Continuando..." -ForegroundColor Cyan
    }

    # Permissões NTFS
    Write-Host "Aplicando permissões NTFS..." -ForegroundColor Yellow
    icacls $pasta /grant "$(GrupoEveryone):(OI)(CI)F" /T
    icacls $pasta /grant "$($usuario):(OI)(CI)F" /T

    # Compartilhamento SMB
    Write-Host "Configurando compartilhamento..." -ForegroundColor Yellow

    if (Get-SmbShare -Name $share -ErrorAction SilentlyContinue) {
        Remove-SmbShare -Name $share -Force
    }

    New-SmbShare -Name $share -Path $pasta -FullAccess $GrupoEveryone, $usuario

    Write-Host "Compartilhamento criado com sucesso!"
    Write-Host "Acesse: \\$(hostname)\Scanner" -ForegroundColor Green
}

# --- Função para executar tudo ---
function FazerTudo {
    Write-Host "Executando configuração completa..." -ForegroundColor Cyan
    Ativar-SMB1
    Criar-UsuarioAdmin -NomeUsuario "scanner"
    Criar-PastaCompartilhada
    Write-Host "Todas as tarefas foram concluídas!" -ForegroundColor Green
}

# ==============================
# MENU INTERATIVO
# ==============================

while ($true) {
    Clear-Host
    Write-Host "===== MENU DE CONFIGURAÇÃO WINDOWS =====" -ForegroundColor Cyan
    Write-Host "1) Executar tudo (Ativar SMB + Usuário admin + Pasta compartilhada)"
    Write-Host "2) Criar usuário administrador"
    Write-Host "3) Criar pasta e compartilhamento"
    Write-Host "4) Ativar SMB 1.0"
    Write-Host "0) Sair"
    Write-Host "========================================"

    $opcao = Read-Host "Escolha uma opção"

    switch ($opcao) {
        "1" { FazerTudo; Pause }
        "2" { Criar-UsuarioAdmin; Pause }
        "3" { Criar-PastaCompartilhada; Pause }
        "4" { Ativar-SMB1; Pause }
        "0" { exit }
        default { Write-Host "Opção inválida!" -ForegroundColor Red; Pause }
    }
}
