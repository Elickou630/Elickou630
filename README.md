import tkinter as tk
from tkinter import messagebox
import os
import pickle

def criar_pasta():
    if not os.path.exists("dados"):
        os.makedirs("dados")

def salvar_dados(comercializacao):
    criar_pasta()
    with open("dados/produtos.pkl", "wb") as f:
        pickle.dump(comercializacao.produtos, f)
    with open("dados/transacoes.pkl", "wb") as f:
        pickle.dump(comercializacao.transacoes, f)
    with open("dados/despesas.pkl", "wb") as f:
        pickle.dump(comercializacao.despesas, f)

def carregar_dados(comercializacao):
    if os.path.exists("dados/produtos.pkl"):
        with open("dados/produtos.pkl", "rb") as f:
            comercializacao.produtos = pickle.load(f)
    if os.path.exists("dados/transacoes.pkl"):
        with open("dados/transacoes.pkl", "rb") as f:
            comercializacao.transacoes = pickle.load(f)
    if os.path.exists("dados/despesas.pkl"):
        with open("dados/despesas.pkl", "rb") as f:
            comercializacao.despesas = pickle.load(f)

class Produto:
    def __init__(self, nome, preco_compra, preco_venda, quantidade_estoque):
        self.nome = nome
        self.preco_compra = preco_compra
        self.preco_venda = preco_venda
        self.quantidade_estoque = quantidade_estoque

    def calcular_lucro_prejuizo(self):
        return self.preco_venda - self.preco_compra

    def __str__(self):
        return f"{self.nome}: {self.quantidade_estoque}"

class Despesa:
    def __init__(self, valor, categoria, descricao):
        self.valor = valor
        self.categoria = categoria
        self.descricao = descricao

class Comercializacao:
    def __init__(self):
        self.produtos = []
        self.transacoes = []
        self.despesas = []
        self.imposto = 0.1  # Taxa de imposto de %

    def adicionar_produto(self, produto):
        self.produtos.append(produto)

    def vender_produto(self, nome, quantidade):
        for produto in self.produtos:
            if produto.nome == nome:
                if produto.quantidade_estoque >= quantidade:
                    produto.quantidade_estoque -= quantidade
                    lucro_transacao = produto.calcular_lucro_prejuizo() * quantidade
                    valor_venda = produto.preco_venda * quantidade
                    valor_imposto = valor_venda * self.imposto
                    self.transacoes.append((produto.nome, quantidade, valor_venda, valor_imposto))
                    return f"{quantidade} unidades de {nome} vendidas com sucesso!"
                else:
                    return f"Quantidade insuficiente de {nome} em estoque."
        return f"{nome} não encontrado no estoque."

    def calcular_lucro_total(self):
        lucro_total = sum(produto.calcular_lucro_prejuizo() * produto.quantidade_estoque for produto in self.produtos)
        return lucro_total

    def calcular_imposto_total(self):
        imposto_total = sum(transacao[3] for transacao in self.transacoes)
        return imposto_total

class GUI:
    def __init__(self, root):
        self.comercializacao = Comercializacao()
        # Carregar dados ao iniciar o aplicativo
        carregar_dados(self.comercializacao)

        self.root = root
        self.root.title("Sistema de Comercialização")
        
        self.frame = tk.Frame(self.root)
        self.frame.pack(padx=10, pady=10)

        self.label_estoque = tk.Label(self.frame, text="Estoque:")
        self.label_estoque.grid( row=0, column=0, padx=5, pady=5)

        self.botao_estoque = tk.Button(self.frame, text="Visualizar Estoque", command=self.visualizar_estoque)
        self.botao_estoque.grid( row=1, column=0, padx=5, pady=5)

        self.botao_venda = tk.Button(self.frame, text="Vender Produto", command=self.vender_produto_window)
        self.botao_venda.grid( row=1, column=1, padx=5, pady=5)

        self.botao_relatorio = tk.Button(self.frame, text="Relatórios", command=self.abrir_relatorios)
        self.botao_relatorio.grid( row=2, column=0, padx=5, pady=5)

        self.botao_cadastrar = tk.Button(self.frame, text="Cadastrar Produtos", command=self.cadastrar_produtos)
        self.botao_cadastrar.grid( row=2, column=1, padx=5, pady=5)

        self.botao_configurar_imposto = tk.Button(self.frame, text="Configurar Imposto", command=self.abrir_configurar_imposto)
        self.botao_configurar_imposto.grid( row=3, column=0, padx=5, pady=5)

        self.botao_pesquisar = tk.Button(self.frame, text="Pesquisar Produto", command=self.pesquisar_produto_window)
        self.botao_pesquisar.grid( row=3, column=1, padx=5, pady=5)
        
        self.botao_despesas = tk.Button(self.frame, text="Despesas", command=self.abrir_despesas)
        self.botao_despesas.grid( row=4, column=0, padx=5, pady=5)
        
        self.botao_salvar = tk.Button(self.frame, text="Salvar", command=lambda: salvar_dados(self.comercializacao))
        self.botao_salvar.grid( row=4, column=1, padx=5, pady=5)

        self.botao_sair = tk.Button(self.frame, text="Sair", command=root.quit)
        self.botao_sair.grid( row=9, column=0, padx=5, pady=5)
        
        self.adicionar_window = None

    def visualizar_estoque(self):
        estoque_str = "Estoque:\n"
        for produto in self.comercializacao.produtos:
            estoque_str += str(produto) + "\n"
        messagebox.showinfo("Estoque", estoque_str)

    def vender_produto_window(self):
        venda_window = tk.Toplevel(self.root)
        venda_window.title("Vender Produto")

        tk.Label(venda_window, text="Nome do Produto:").grid(row=0, column=0)
        tk.Label(venda_window, text="Quantidade:").grid(row=1, column=0)

        self.nome_entry = tk.Entry(venda_window)
        self.nome_entry.grid(row=0, column=1)
        self.quantidade_entry = tk.Entry(venda_window)
        self.quantidade_entry.grid(row=1, column=1)

        tk.Button(venda_window, text="Confirmar", command=self.confirmar_venda).grid(row=2, column=0, columnspan=2)

    def confirmar_venda(self):
        nome = self.nome_entry.get()
        quantidade = int(self.quantidade_entry.get())
        resultado = self.comercializacao.vender_produto(nome, quantidade)
        messagebox.showinfo("Resultado da Venda", resultado)

    def abrir_relatorios(self):
        relatorios_window = tk.Toplevel(self.root)
        relatorios_window.title("Relatórios")

        tk.Button(relatorios_window, text="Relatório de Lucro das Vendas", command=self.relatorio_lucro_vendas).grid(row=0, column=0, padx=5, pady=5)
        tk.Button(relatorios_window, text="Relatório Geral", command=self.relatorio_geral).grid(row=1, column=0, padx=5, pady=5)

    def relatorio_lucro_vendas(self):
        lucro_total = self.comercializacao.calcular_lucro_total()
        imposto_total = self.comercializacao.calcular_imposto_total()
        lucro_liquido = lucro_total - imposto_total
        relatorio_str = f"Lucro Total: MZN { lucro_total:.2f}\nImposto Total: MZN { imposto_total:.2f}\nLucro Líquido: MZN { lucro_liquido:.2f}"
        messagebox.showinfo("Relatório de Lucro das Vendas", relatorio_str)

    def relatorio_geral(self):
        relatorio_str = "Relatório Geral:\n"
        for produto in self.comercializacao.produtos:
            custo_compra = produto.preco_compra * produto.quantidade_estoque
            custo_venda = produto.preco_venda * produto.quantidade_estoque
            relatorio_str += f"Produto: { produto.nome}\nQuantidade: { produto.quantidade_estoque}\nCusto da Compra: MZN { custo_compra:.2f}\nCusto da Venda: MZN { custo_venda:.2f}\n\n"
        messagebox.showinfo("Relatório Geral", relatorio_str)

    def cadastrar_produtos(self):
        cadastrar_window = tk.Toplevel(self.root)
        cadastrar_window.title("Cadastrar Produto")

        tk.Button(cadastrar_window, text="Adicionar Produto", command=self.adicionar_produto_window).grid(row=0, column=0, padx=5, pady=5)
        tk.Button(cadastrar_window, text="Alterar Produto", command=self.alterar_produto_window).grid(row=1, column=0, padx=5, pady=5)
        tk.Button(cadastrar_window, text="Apagar Produto", command=self.apagar_produto_window).grid(row=2, column=0, padx=5, pady=5)

    def adicionar_produto_window(self):
        self.adicionar_window = tk.Toplevel(self.root)
        self.adicionar_window.title("Adicionar Produto")

        tk.Label(self.adicionar_window, text="Nome do Produto:").grid(row=0, column=0)
        tk.Label(self.adicionar_window, text="Preço de Compra:").grid(row=1, column=0)
        tk.Label(self.adicionar_window, text="Preço de Venda:").grid(row=2, column=0)
        tk.Label(self.adicionar_window, text="Quantidade em Estoque:").grid(row=3, column=0)

        self.nome_entry = tk.Entry(self.adicionar_window)
        self.nome_entry.grid(row=0, column=1)
        self.preco_compra_entry = tk.Entry(self.adicionar_window)
        self.preco_compra_entry.grid(row=1, column=1)
        self.preco_venda_entry = tk.Entry(self.adicionar_window)
        self.preco_venda_entry.grid(row=2, column=1)
        self.quantidade_estoque_entry = tk.Entry(self.adicionar_window)
        self.quantidade_estoque_entry.grid(row=3, column=1)

        tk.Button(self.adicionar_window, text="Confirmar", command=self.confirmar_adicao).grid(row=4, column=0, columnspan=2)

    def confirmar_adicao(self):
        nome = self.nome_entry.get()
        preco_compra_str = self.preco_compra_entry.get()
        preco_venda_str = self.preco_venda_entry.get()
        quantidade_estoque_str = self.quantidade_estoque_entry.get()
        if preco_compra_str.strip() and preco_venda_str.strip() and quantidade_estoque_str.strip():
            preco_compra = float(preco_compra_str)
            preco_venda = float(preco_venda_str)
            quantidade_estoque = int(quantidade_estoque_str)
            produto = Produto(nome, preco_compra, preco_venda, quantidade_estoque)
            self.comercializacao.adicionar_produto(produto)
            messagebox.showinfo("Sucesso", "Produto adicionado com sucesso!")
            if self.adicionar_window:
                self.adicionar_window.destroy()
        else:
            messagebox.showerror("Erro", "Por favor, preencha todos os campos.")

    def alterar_produto_window(self):
        alterar_window = tk.Toplevel(self.root)
        alterar_window.title("Alterar Produto")

        tk.Label(alterar_window, text="Nome do Produto:").grid(row=0, column=0)
        self.nome_produto_entry = tk.Entry(alterar_window)
        self.nome_produto_entry.grid(row=0, column=1)

        tk.Label(alterar_window, text="Nova Quantidade em Estoque:").grid(row=1, column=0)
        self.nova_quantidade_estoque_entry = tk.Entry(alterar_window)
        self.nova_quantidade_estoque_entry.grid(row=1, column=1)

        tk.Button(alterar_window, text="Confirmar", command=self.confirmar_alteracao).grid(row=2, column=0, columnspan=2)

    def confirmar_alteracao(self):
        nome = self.nome_produto_entry.get()
        nova_quantidade_estoque_str = self.nova_quantidade_estoque_entry.get()
        if nome.strip() and nova_quantidade_estoque_str.strip():
            nova_quantidade_estoque = int(nova_quantidade_estoque_str)
            for produto in self.comercializacao.produtos:
                if produto.nome == nome:
                    produto.quantidade_estoque = nova_quantidade_estoque
                    messagebox.showinfo("Sucesso", "Produto alterado com sucesso!")
                    return
            messagebox.showerror("Erro", f"Produto '{nome}' não encontrado.")
        else:
            messagebox.showerror("Erro", "Por favor, preencha todos os campos.")

    def apagar_produto_window(self):
        apagar_window = tk.Toplevel(self.root)
        apagar_window.title("Apagar Produto")

        tk.Label(apagar_window, text="Nome do Produto:").grid(row=0, column=0)
        self.nome_produto_entry = tk.Entry(apagar_window)
        self.nome_produto_entry.grid(row=0, column=1)

        tk.Button(apagar_window, text="Confirmar", command=self.confirmar_apagar).grid(row=1, column=0, columnspan=2)

    def confirmar_apagar(self):
        nome = self.nome_produto_entry.get()
        if nome.strip():
            for produto in self.comercializacao.produtos:
                if produto.nome == nome:
                    self.comercializacao.produtos.remove(produto)
                    messagebox.showinfo("Sucesso", "Produto apagado com sucesso!")
                    return
            messagebox.showerror("Erro", f"Produto '{nome}' não encontrado.")
        else:
            messagebox.showerror("Erro", "Por favor, preencha todos os campos.")

    def abrir_configurar_imposto(self):
        imposto_window = tk.Toplevel(self.root)
        imposto_window.title("Configurar Imposto")

        tk.Label(imposto_window, text="Taxa de Imposto (%):").grid(row=0, column=0)
        self.imposto_entry = tk.Entry(imposto_window)
        self.imposto_entry.grid(row=0, column=1)

        tk.Button(imposto_window, text="Confirmar", command=self.confirmar_imposto).grid(row=1, column=0, columnspan=2)

    def confirmar_imposto(self):
        imposto_str = self.imposto_entry.get()
        if imposto_str.strip():
            imposto = float(imposto_str)
            self.comercializacao.imposto = imposto / 100
            messagebox.showinfo("Sucesso", f"Taxa de imposto atualizada para {imposto}%.")
        else:
            messagebox.showerror("Erro", "Por favor, preencha o campo.")

    def abrir_despesas(self):
        despesas_window = tk.Toplevel(self.root)
        despesas_window.title("Despesas")

        tk.Button(despesas_window, text="Nova Despesa", command=self.nova_despesa).grid(row=0, column=0, padx=5, pady=5)
        tk.Button(despesas_window, text="Alterar Despesa", command=self.alterar_despesa).grid(row=1, column=0, padx=5, pady=5)
        tk.Button(despesas_window, text="Apagar Despesa", command=self.apagar_despesa).grid(row=2, column=0, padx=5, pady=5)
        
        tk.Button(despesas_window, text="Visualizar Despesas", command=self.visualizar_despesas).grid(row=3, column=0, padx=5, pady=5)

    def nova_despesa(self):
        nova_despesa_window = tk.Toplevel(self.root)
        nova_despesa_window.title("Nova Despesa")

        tk.Label(nova_despesa_window, text="Valor da Despesa:").grid(row=0, column=0)
        tk.Label(nova_despesa_window, text="Categoria da Despesa:").grid(row=1, column=0)
        tk.Label(nova_despesa_window, text="Descrição:").grid(row=2, column=0)

        self.valor_despesa_entry = tk.Entry(nova_despesa_window)
        self.valor_despesa_entry.grid(row=0, column=1)
        self.categoria_despesa_entry = tk.Entry(nova_despesa_window)
        self.categoria_despesa_entry.grid(row=1, column=1)
        self.descricao_despesa_entry = tk.Entry(nova_despesa_window)
        self.descricao_despesa_entry.grid(row=2, column=1)

        tk.Button(nova_despesa_window, text="Confirmar", command=self.confirmar_nova_despesa).grid(row=3, column=0, columnspan=2)

    def confirmar_nova_despesa(self):
        valor_despesa_str = self.valor_despesa_entry.get()
        categoria_despesa = self.categoria_despesa_entry.get()
        descricao_despesa = self.descricao_despesa_entry.get()
        if valor_despesa_str.strip() and categoria_despesa.strip() and descricao_despesa.strip():
            valor_despesa = float(valor_despesa_str)
            despesa = Despesa(valor_despesa, categoria_despesa, descricao_despesa)
            self.comercializacao.despesas.append(despesa)
            messagebox.showinfo("Sucesso", "Despesa adicionada com sucesso!")
        else:
            messagebox.showerror("Erro", "Por favor, preencha todos os campos.")

    def alterar_despesa(self):
        alterar_despesa_window = tk.Toplevel(self.root)
        alterar_despesa_window.title("Alterar Despesa")

        tk.Label(alterar_despesa_window, text="Selecione a Despesa:").grid(row=0, column=0)
        despesa_var = tk.StringVar(alterar_despesa_window)
        despesa_var.set("")  # Valor inicial vazio
        despesa_option_menu = tk.OptionMenu(alterar_despesa_window, despesa_var, *[""] + [f"{despesa.categoria}: {despesa.descricao}" for despesa in self.comercializacao.despesas])
        despesa_option_menu.grid(row=0, column=1)

        tk.Button(alterar_despesa_window, text="Selecionar", command=lambda: self.abrir_janela_alterar_despesa(despesa_var.get())).grid(row=1, column=0, columnspan=2)

    def abrir_janela_alterar_despesa(self, despesa_selecionada):
        if despesa_selecionada:
            categoria, descricao = despesa_selecionada.split(": ")
            for despesa in self.comercializacao.despesas:
                if despesa.categoria == categoria and despesa.descricao == descricao:
                    alterar_despesa_window = tk.Toplevel(self.root)
                    alterar_despesa_window.title("Alterar Despesa")

                    tk.Label(alterar_despesa_window, text="Valor da Despesa:").grid(row=0, column=0)
                    tk.Label(alterar_despesa_window, text="Nova Categoria da Despesa:").grid( row=1, column=0)
                    tk.Label(alterar_despesa_window, text="Nova Descrição:").grid( row=2, column=0)

                    self.novo_valor_despesa_entry = tk.Entry(alterar_despesa_window)
                    self.novo_valor_despesa_entry.grid( row=0, column=1)
                    self.nova_categoria_despesa_entry = tk.Entry(alterar_despesa_window)
                    self.nova_categoria_despesa_entry.grid( row=1, column=1)
                    self.nova_descricao_despesa_entry = tk.Entry(alterar_despesa_window)
                    self.nova_descricao_despesa_entry.grid( row=2, column=1)

                    tk.Button(alterar_despesa_window, text= "Confirmar", command=lambda: self.confirmar_alteracao_despesa(despesa, alterar_despesa_window)).grid( row=3, column=0, columnspan=2)
                    return
        messagebox.showerror("Erro", "Por favor, selecione uma despesa.")

    def confirmar_alteracao_despesa(self, despesa, alterar_despesa_window):
        novo_valor_despesa_str = self.novo_valor_despesa_entry.get()
        nova_categoria_despesa = self.nova_categoria_despesa_entry.get()
        nova_descricao_despesa = self.nova_descricao_despesa_entry.get()
        if novo_valor_despesa_str.strip() or nova_categoria_despesa.strip() or nova_descricao_despesa.strip():
            if novo_valor_despesa_str.strip():
                despesa.valor = float(novo_valor_despesa_str)
            if nova_categoria_despesa.strip():
                despesa.categoria = nova_categoria_despesa
            if nova_descricao_despesa.strip():
                despesa.descricao = nova_descricao_despesa
            messagebox.showinfo("Sucesso", "Despesa alterada com sucesso!")
            alterar_despesa_window.destroy()
        else:
            messagebox.showerror("Erro", "Por favor, preencha pelo menos um campo.")

    def apagar_despesa(self):
        apagar_despesa_window = tk.Toplevel(self.root)
        apagar_despesa_window.title("Apagar Despesa")

        tk.Label(apagar_despesa_window, text="Selecione a Despesa:").grid(row=0, column=0)
        despesa_var = tk.StringVar(apagar_despesa_window)
        despesa_var.set("")  # Valor inicial vazio
        despesa_option_menu = tk.OptionMenu(apagar_despesa_window, despesa_var, *[""] + [f"{despesa.categoria}: {despesa.descricao}" for despesa in self.comercializacao.despesas])
        despesa_option_menu.grid(row=0, column=1)

        tk.Button(apagar_despesa_window, text="Apagar", command=lambda: self.confirmar_apagar_despesa(despesa_var.get())).grid(row=1, column=0, columnspan=2)

    def confirmar_apagar_despesa(self, despesa_selecionada):
        if despesa_selecionada:
            categoria, descricao = despesa_selecionada.split(": ")
            for despesa in self.comercializacao.despesas:
                if despesa.categoria == categoria and despesa.descricao == descricao:
                    self.comercializacao.despesas.remove(despesa)
                    messagebox.showinfo("Sucesso", "Despesa apagada com sucesso!")
                    return
        messagebox.showerror("Erro", "Por favor, selecione uma despesa.")

    def visualizar_despesas(self):
        despesas_str = "Despesas:\n"
        for despesa in self.comercializacao.despesas:
            despesas_str += f"Valor: {despesa.valor}, Categoria: {despesa.categoria}, Descrição: {despesa.descricao}\n"
        if despesas_str == "Despesas:\n":
            despesas_str += "Nenhuma despesa cadastrada."
        messagebox.showinfo("Despesas", despesas_str)
    
    def pesquisar_produto_window(self):
        pesquisar_window = tk.Toplevel(self.root)
        pesquisar_window.title("Pesquisar Produto")

        tk.Label(pesquisar_window, text="Nome do Produto:").grid(row=0, column=0)
        self.nome_pesquisa_entry = tk.Entry(pesquisar_window)
        self.nome_pesquisa_entry.grid(row=0, column=1)

        tk.Button(pesquisar_window, text="Pesquisar", command=self.pesquisar_produto).grid(row=1, column=0, columnspan=2)

    def pesquisar_produto(self):
        nome_pesquisa = self.nome_pesquisa_entry.get()
        for produto in self.comercializacao.produtos:
            if produto.nome == nome_pesquisa:
                messagebox.showinfo("Produto Encontrado", f"Nome: {produto.nome}\nPreço de Compra: {produto.preco_compra}\nPreço de Venda: {produto.preco_venda}\nQuantidade em Estoque: {produto.quantidade_estoque}")
                return
        messagebox.showerror("Erro", "Produto não encontrado.")

if __name__ == "__main__":
    root = tk.Tk()
    gui = GUI(root)
    root.mainloop()