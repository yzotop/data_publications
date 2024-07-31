---
tags:
  - data
link: https://habr.com/ru/companies/otus/articles/799865/
source: habr
data_type:
  - DS
---




_Привет, Хабр!_

**Actor-Critic** — это класс алгоритмов в RL, суть которого довольно проста на словах, он сочетает в себе такие политики как _policy-based_ и оценки _value-based_. У нас есть два главных действующих лица: _Actor_ и _Critic_**. Actor отвечает за выбор действий,** т.е формирование политики поведения, он принимает решения исходя из текущего состояния окружающей среды. **Critic оценивает, насколько хорошо или плохо Actor справляется со своей задачей**, предоставляя обратную связь через оценку действий Actor'a.

#### Начнем с Actor

Actor отвечает за выбор действий, которые агент должен выполнить в данном состоянии окружающей среды. Actor принимает текущее состояние окружения как входные данные и генерирует действия как выходные данных, это можно реализовать с помощью аппроксимации.

Реализуем просто Actor, который использует нейронную сеть для генерации действий, сделаем это с помощью TensorFlow:

```
import tensorflow as tffrom tensorflow.keras import layersclass SimpleActorModel(tf.keras.Model):    def __init__(self, action_size):        super(SimpleActorModel, self).__init__()        self.dense1 = layers.Dense(128, activation="relu")        self.dense2 = layers.Dense(128, activation="relu")        self.output_action = layers.Dense(action_size, activation="softmax")    def call(self, state):        x = self.dense1(state)        x = self.dense2(x)        return self.output_action(x)# используемaction_size = 2 # предположим, есть 2 возможных действияmodel = SimpleActorModel(action_size)state = tf.constant([[0.1, 0.2, 0.3]]) # пример состояния окружающей средыaction_probabilities = model(state)print("Распределение вероятностей действий:", action_probabilities.numpy())
```

`SimpleActorModel` принимает на вход состояние окружающей среды, в нашем случае это вектор и выводит распределение вероятностей по возможным действиям. Здесь используется два скрытых слоя с активацией ReLU для обработки состояния, а на выходе — softmax, чтобы получить вероятности для каждого действия.

#### Critic

Critic оценивает, насколько хорошо или плохо выбранное действие Actor'а с точки зрения достижения конечной цели. Это делается путем расчета value function, которая оценивает будущие награды, получаемые за определенное действие или находясь в определенном состоянии. Critic пытается предсказать, какое вознаграждение агент может ожидать в долгосрочной перспективе, принимая во внимание текущее состояние и предпринимаемое действие.

Для реализации Critic можно использовать можно использовать TensorFlow:

```
import tensorflow as tffrom tensorflow.keras import layersclass SimpleCriticModel(tf.keras.Model):    def __init__(self):        super(SimpleCriticModel, self).__init__()        self.dense1 = layers.Dense(128, activation="relu")        self.dense2 = layers.Dense(128, activation="relu")        self.value = layers.Dense(1)    def call(self, state):        x = self.dense1(state)        x = self.dense2(x)        return self.value(x)# пример использованияmodel = SimpleCriticModel()state = tf.constant([[0.1, 0.2, 0.3]]) # пример состояния окружающей средыvalue = model(state)print("Предсказанная ценность состояния:", value.numpy())
```

`SimpleCriticModel` анализирует текущее состояние среды и возвращает оценку ценности этого состояния, Critic выдает числовое значение, представляющее ожидаемую награду.

## Actor-Critic алгоритм

И вот **Actor-Critic** объединяет два компонента.

Actor выбирает действие на основе текущего состояния среды.

После выполнения действия, Critic оценивает его, используя функцию ценности. Эта оценка показывает, насколько выбранное действие было хорошо с точки зрения ожидаемого долгосрочного вознаграждения.

На основе оценки Critic, Actor корректирует свою стратегию действий. Это может быть реализовано, например, через механизм _градиентного восхождения_ по политике, где Actor обновляет свои параметры в направлении, увеличивающем ожидаемое вознаграждение.

Critic также обновляет свои параметры на основе полученного вознаграждения и разницы между ожидаемым и полученным вознаграждениями, чтобы в будущем делать более точные предсказания.

#### Примеры реализации

Базовый пример с PyTorch:

```
import torchimport torch.nn as nnimport torch.optim as optimclass Actor(nn.Module):    def __init__(self, input_dim, output_dim):        super(Actor, self).__init__()        self.linear = nn.Linear(input_dim, output_dim)        def forward(self, state):        return torch.softmax(self.linear(state), dim=-1)class Critic(nn.Module):    def __init__(self, input_dim):        super(Critic, self).__init__()        self.linear = nn.Linear(input_dim, 1)        def forward(self, state):        return self.linear(state)def train(actor, critic, state, action, reward, next_state, done):    optimizer_actor = optim.Adam(actor.parameters(), lr=1e-3)    optimizer_critic = optim.Adam(critic.parameters(), lr=1e-3)        # Critic update    value = critic(state)    next_value = critic(next_state)    td_error = reward + (1 - done) * 0.99 * next_value - value    critic_loss = td_error.pow(2)    optimizer_critic.zero_grad()    critic_loss.backward()    optimizer_critic.step()    # Actor update    log_prob = torch.log(actor(state)[action])    actor_loss = -log_prob * td_error.detach()    optimizer_actor.zero_grad()    actor_loss.backward()    optimizer_actor.step()
```

Реализация на TensorFlow:

```
import tensorflow as tffrom tensorflow.keras import layersclass ActorCritic(tf.keras.Model):    def __init__(self, num_actions):        super(ActorCritic, self).__init__()        self.common = layers.Dense(128, activation='relu')        self.actor = layers.Dense(num_actions, activation='softmax')        self.critic = layers.Dense(1)    def call(self, inputs):        x = self.common(inputs)        return self.actor(x), self.critic(x)model = ActorCritic(num_actions=4)optimizer = tf.keras.optimizers.Adam(lr=0.01)def train_step(state, action, reward, next_state, done):    with tf.GradientTape() as tape:        action_probs, critic_value = model(state)        _, critic_value_next = model(next_state)        action_log_probs = tf.math.log(action_probs[0, action])        td_error = reward + 0.99 * critic_value_next * (1 - done) - critic_value        actor_loss = -action_log_probs * tf.stop_gradient(td_error)        critic_loss = td_error**2    grads = tape.gradient([actor_loss, critic_loss], model.trainable_variables)    optimizer.apply_gradients(zip(grads, model.trainable_variables))
```

TensorFlow и Keras с Functional API:

```
import tensorflow as tffrom tensorflow.keras.layers import Input, Densefrom tensorflow.keras.models import Modeldef create_actor(input_shape, output_shape):    inputs = Input(shape=input_shape)    x = Dense(64, activation='relu')(inputs)    outputs = Dense(output_shape, activation='softmax')(x)    model = Model(inputs, outputs)    return modeldef create_critic(input_shape):    inputs = Input(shape=input_shape)    x = Dense(64, activation='relu')(inputs)    outputs = Dense(1)(x)    model = Model(inputs, outputs)    return modelactor = create_actor(input_shape=(4,), output_shape=2)critic = create_critic(input_shape=(4,))
```

PyTorch с параллельным обновленеим:

```
import torchimport torch.nn as nnimport torch.optim as optimclass ActorCritic(nn.Module):    def __init__(self, input_dim, action_dim):        super(ActorCritic, self).__init__()        self.common = nn.Linear(input_dim, 64)        self.actor = nn.Linear(64, action_dim)        self.critic = nn.Linear(64, 1)        def forward(self, state):        x = torch.relu(self.common(state))        return torch.softmax(self.actor(x), dim=-1), self.critic(x)def train(model, state, action, reward, next_state, done):    optimizer = optim.Adam(model.parameters(), lr=5e-4)        # Compute loss    action_probs, value = model(state)    _, next_value = model(next_state)    td_error = reward + (1 - done) * 0.99 * next_value - value    action_log_probs = torch.log(action_probs[action])    actor_loss = -action_log_probs * td_error.detach()    critic_loss = td_error.pow(2)    # Optimize the model    optimizer.zero_grad()    (actor_loss + critic_loss).backward()    optimizer.step()
```

#### Advantage Actor-Critic

В Advantage Actor-Critic акцент делается на балансе между оценкой текущих действий actor и критической оценкой этих действий critic, используя функцию преимущества.

_(Advantage Function, A(s,a))_ вычисляет разницу между ожидаемой наградой за выбранное действие и средней ожидаемой наградой в данном состоянии. Формально, _A(s,a) = Q(s,a) - V(s)_, где _Q(s,a)_ — это ожидаемая награда за действие _a_ в состоянии _s_, а _V(s)_ — это ожидаемая награда в состоянии _s_, не зависящая от действия. Это позволяет actor обновлять политику в направлении увеличения преимущества, то есть выбирать действия, которые работают лучше, чем среднее, для данного состояния.

**Asynchronous Advantage Actor-Critic (A3C)**: один из наиболее известных вариантов A2C, где несколько акторов параллельно исследуют пространство и обновляют глобальную модель асинхронно, пример:

```
import torchimport torch.nn as nnimport torch.optim as optimimport numpy as npimport threadingimport gymclass ActorCritic(nn.Module):    def __init__(self, num_inputs, num_actions):        super(ActorCritic, self).__init__()        self.critic = nn.Linear(num_inputs, 1)        self.actor = nn.Linear(num_inputs, num_actions)        def forward(self, x):        value = self.critic(x)        probs = torch.softmax(self.actor(x), dim=-1)        return probs, valuedef worker(global_model, optimizer, env_name, global_results, lock, worker_id):    local_model = ActorCritic(num_inputs, num_actions)    local_model.load_state_dict(global_model.state_dict())        env = gym.make(env_name)    state = env.reset()        while True:        # код для выполнения действий агентом и обновления модели                with lock:            global_model.load_state_dict(local_model.state_dict())# глобал инициализация и запуск воркеровglobal_model = ActorCritic(num_inputs, num_actions)optimizer = optim.Adam(global_model.parameters())workers = [threading.Thread(target=worker, args=(global_model, optimizer, 'CartPole-v1', global_results, lock, i)) for i in range(num_workers)]for w in workers:    w.start()for w in workers:    w.join()
```

**Synchronous Advantage Actor-Critic (Synchronous A2C или A2C)**: в отличие от A3C, A2C обновляет глобальную модель синхронно, используя средние значения градиентов от всех акторов, это в какой-то мере уменьшает дисперсию обновлений, хотя потенциально может быть медленнее по сравнению с асинхронным подходом. Пример:

```
import torchimport torch.nn as nnimport torch.optim as optimimport numpy as npimport gymclass ActorCritic(nn.Module):    # конструктор и forward метод аналогичны A3Cdef update_global(optimizer, global_model, loss):    optimizer.zero_grad()    loss.backward()    optimizer.step()env = gym.make('CartPole-v1')global_model = ActorCritic(num_inputs, num_actions)optimizer = optim.Adam(global_model.parameters())# синхронное обновлениеfor episode in range(total_episodes):    state = env.reset()    done = False    while not done:        # выполнение действий и сбор градиентов для обновления        loss = compute_loss(...)  # функция вычисления потерь на основе действий и оценок        update_global(optimizer, global_model, loss)
```

**Proximal Policy Optimization (PPO)**: хотя не является прямым вариантом A2C, PPO развивает идеи Actor-Critic методов, вводя функцию потерь, которая ограничивает изменения в политике:

```
import torchimport torch.nn as nnimport torch.optim as optimimport numpy as npimport gymclass ActorCritic(nn.Module):    def __init__(self, num_inputs, num_actions):        super(ActorCritic, self).__init__()        self.critic = nn.Linear(num_inputs, 1)        self.actor = nn.Linear(num_inputs, num_actions)        def forward(self, x):        value = self.critic(x)        probs = torch.softmax(self.actor(x), dim=-1)        return probs, valuedef ppo_loss(old_log_probs, new_log_probs, advantages, clip_param=0.2):    ratios = torch.exp(new_log_probs - old_log_probs)    surr1 = ratios * advantages    surr2 = torch.clamp(ratios, 1.0 - clip_param, 1.0 + clip_param) * advantages    return -torch.min(surr1, surr2).mean()def compute_advantages(rewards, values, gamma=0.99, tau=0.95):    advantages = torch.zeros_like(rewards)    gae = 0    for t in reversed(range(len(rewards))):        delta = rewards[t] + gamma * values[t + 1] - values[t]        gae = delta + gamma * tau * gae        advantages[t] = gae    return advantages# инициализация среды, модели и оптимизатораenv = gym.make('CartPole-v1')num_inputs = env.observation_space.shape[0]num_actions = env.action_space.nmodel = ActorCritic(num_inputs, num_actions)optimizer = optim.Adam(model.parameters(), lr=1e-3)for epoch in range(10):  # Пример количества эпох обучения    # реализация сбора данных среды, выполнения действий и т.д.        # предполагается, что мы уже собрали данные о состояниях, действиях, наградах    # states, actions, rewards, next_states, dones        # вычисление логарифмов вероятностей, преимуществ, функции потерь и выполнение шага оптимизации    old_log_probs = ...  # логарифмы старых вероятностей действий    new_log_probs = ...  # ллогарифмы новых вероятностей действий, полученных из модели    advantages = ...  # преимущества, вычисленные с использованием функции compute_advantages        loss = ppo_loss(old_log_probs, new_log_probs, advantages)    optimizer.zero_grad()    loss.backward()    optimizer.step()
```

---

В завершение хочу пригласить вас на бесплатный вебинар про применение фреймворка FinRL для моделирования торгового агента. [Регистрация доступна по ссылке](https://otus.pw/8zeT/).