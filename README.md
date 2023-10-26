# A
VIEW
def mostrar_alunos(request):
    if request.method == 'GET':
        form = FiltroAlunoForm(request.GET)
        alunos = Aluno.objects.all()
        if form.is_valid():
            if form.cleaned_data['nome']:
                alunos = alunos.filter(nome_aluno__icontains=form.cleaned_data['nome'])
            if form.cleaned_data['cidade']:
                alunos = alunos.filter(cidade=form.cleaned_data['cidade'])
            if form.cleaned_data['curso']:
                alunos = alunos.filter(curso=form.cleaned_data['curso'])
        paginator = Paginator(alunos, 5)  # Show 25 contacts per page.

        page_number = request.GET.get("page")
        page_obj = paginator.get_page(page_number)
    else:
        form = FiltroAlunoForm()
        alunos = Aluno.objects.all()

    return render(request, 'aluno/alunos.html', {"page_obj": page_obj, 'form': form})

FORM
    class FiltroAlunoForm(forms.Form):
    nome = forms.CharField(
        max_length=150,
        required=False,
        label='Nome',
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    cidade = forms.ModelChoiceField(
        queryset=Cidade.objects.all(),
        required=False,
        empty_label='Todas as cidades',
        label='Cidade',
        widget=forms.Select(attrs={'class': 'form-select'})
    )
    curso = forms.ModelChoiceField(
        queryset=Curso.objects.all(),
        required=False,
        empty_label='Todos os cursos',
        label='Curso',
        widget=forms.Select(attrs={'class': 'form-select'})
    )


    URL 
    from django.contrib import admin
from django.contrib.auth import views as auth_views
from django.urls import path, include
from aluno.views import aluno_criar,index,aluno_editar,aluno_remover,mostrar_alunos

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',index,name='index'),
    path('aluno/',aluno_criar,name='aluno_criar'),
    path('aluno/editar/<int:id>/',aluno_editar, name='aluno_editar'),
    path('aluno/remover/<int:id>/',aluno_remover,name='aluno_remover'),
    path('aluno/mostrar',mostrar_alunos,name='mostrar_alunos'),
]

TEMPLATE
{% extends 'base.html' %}

{% block center %}
    <h2> Lista de Alunos</h2>

    <a href="{% url 'aluno_criar' %}" class="btn btn-success mb-3">Novo Cadastro</a>
    <h1>Filtrar Alunos</h1>
    <form method="get">
            {{ form.as_p }}
        <input type="submit" class="btn btn-primary" value="Filtrar">
    </form>

    <table class="table table-bordered">
        <thead>
        <tr>
            <th>Nome</th>
            <th>Idade</th>
            <th>Email</th>
            <th>Cidade</th>
            <th>Curso</th>
            <th>Ações</th>
        </tr>
        </thead>
        <tbody>
        {% for aluno in page_obj %}
            <tr>
                <td>{{ aluno.nome_aluno }}</td>
                <td>{{ aluno.data_nascimento }}</td>
                <td>{{ aluno.email }}</td>
                <td>{{ aluno.cidade.nome }}</td>
                <td>{{ aluno.curso.nome }}</td>
                <td>
                    <a href="{% url 'aluno_editar' id=aluno.id %}" class="btn btn-primary btn-sm">Editar</a>
                    <a href="{% url 'aluno_remover' id=aluno.id %}" class="btn btn-danger btn-sm">Remover</a>
                </td>
            </tr>
        {% empty %}
            <li>Nenhum aluno encontrado.</li>
        {% endfor %}
        </tbody>
    </table>
    <div class="pagination">
        <span class="step-links">
            {% if page_obj.has_previous %}
                <a href="?page={{ page_obj.previous_page_number }}">Anterior</a>
            {% endif %}
    
            {% for num in page_obj.paginator.page_range %}
                <a class="btn btn-outline-primary rounded-pill" href="?page={{ num }}">{{ num }}</a>
            {% endfor %}
    
            {% if page_obj.has_next %}
                <a href="?page={{ page_obj.next_page_number }}">Próxima</a>
            {% endif %}
        </span>
    </div>


{% endblock %}
