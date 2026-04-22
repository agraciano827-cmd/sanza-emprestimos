<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { 
  getFirestore, collection, addDoc, getDocs, deleteDoc, doc, updateDoc 
} from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCur6eOFsD32uz6EEwT-pfNPii-pKq1AGs",
  authDomain: "sanza-emprestimos.firebaseapp.com",
  projectId: "sanza-emprestimos",
  storageBucket: "sanza-emprestimos.firebasestorage.app",
  messagingSenderId: "789742780283",
  appId: "1:789742780283:web:114b9c40497a561e2007de"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

const lista = document.getElementById("lista");

function formatar(valor){
    return valor.toLocaleString('pt-AO',{
        style:'currency',
        currency:'AOA'
    });
}

async function salvar(){
    let nome = document.getElementById('nome').value;
    let valor = parseFloat(document.getElementById('valor').value);

    if(!nome || !valor) return;

    let juros = valor * 0.35;
    let total = valor + juros;

    await addDoc(collection(db, "clientes"), {
        nome, valor, juros, total
    });

    carregar();
}

window.salvar = salvar;

async function apagar(id){
    await deleteDoc(doc(db, "clientes", id));
    carregar();
}

window.apagar = apagar;

async function editar(id, nomeAtual, valorAtual){
    let novoNome = prompt("Editar nome:", nomeAtual);
    let novoValor = parseFloat(prompt("Editar valor:", valorAtual));

    if(!novoNome || !novoValor) return;

    let juros = novoValor * 0.35;
    let total = novoValor + juros;

    await updateDoc(doc(db, "clientes", id), {
        nome: novoNome,
        valor: novoValor,
        juros,
        total
    });

    carregar();
}

window.editar = editar;

async function carregar(){
    lista.innerHTML = "";

    let emprestado=0, divida=0, lucro=0;

    const querySnapshot = await getDocs(collection(db, "clientes"));

    querySnapshot.forEach((docItem)=>{
        let d = docItem.data();
        let id = docItem.id;

        emprestado += d.valor;
        divida += d.total;
        lucro += d.juros;

        lista.innerHTML += `
        <div class="cliente">
            <div class="nome">${d.nome}</div>
            <div class="info">💰 ${formatar(d.valor)}</div>
            <div class="info">📊 Juros: ${formatar(d.juros)}</div>
            <div class="info">📈 Total: ${formatar(d.total)}</div>

            <button onclick="editar('${id}', '${d.nome}', ${d.valor})">✏️ Editar</button>
            <button class="btn-delete" onclick="apagar('${id}')">🗑 Apagar</button>
        </div>
        `;
    });

    document.getElementById('totalEmprestado').innerText = formatar(emprestado);
    document.getElementById('totalDivida').innerText = formatar(divida);
    document.getElementById('totalLucro').innerText = formatar(lucro);
}

carregar();
</script>
