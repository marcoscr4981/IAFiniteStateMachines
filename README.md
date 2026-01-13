# IA Avanzada FSM

## 01. Crear el plano

He creado un GameObject tipo _Plane_. Se le añade el componente _NavMesh Modifier_.

## 02. Player

Se añade el prefab _Prefabs/FPCharacter_ y se cambia el nombre por **Player**.

## 03. Main Camera

Se desactiva la _Main Camera_ para dejar la cámara que lleva incluida el prefab de Player.

## 04. Enemy

1. Se añade el prefab _Standar Assets/Ugg/Ugg_ y se renombra por **Enemy**.
2. En el componente _Animator_ se añade el controlador _Standar Assets/Ugg/Ugg_.
3. Se añade el componente _Audio Source_ y se incluye el sonido _GuardScene/shoot_.
4. Se añade el script _IA_ y se añade el Game Object **PlayerCapsule** que es hijo del Game Object **Player**.
5. Se añade el componente _Nav Mesh Agent_. No se modifica ninguna propiedad en el **Inspector** ya que están gestionadas con el script _State_.

## 05. Checkpoints

Se añaden 6 Game Objects tipo _Cube_ y se colocan por la escena. Se les añade el tag _Checkpoints_.

## 06. Pared

Se añade un Game Object tipo _Cube_, el cual se amplía y se coloca entre dos Checkpoints para utilizarlo como obstáculo para el NPC.

Se añade el componente _NavMesh Modifier_.

## 07. Crear el NavMesh Surface

He creado un _Empty Object_, le añado el componente _NavMesh Surface_ y después **Bake**.

## 08. Crear un nuevo estado **DIE**

Añado un nuevo estado _Die_. Cuando el jugador hace clic con el botón izquierdo del ratón, el NPC "muere".

Script _State_

### Clase State

Se añade un nuevo estado

```csharp
public enum STATE {

    // ... Resto de estados ...
    DIE         // Morir
};
```

### Clases Idle, Patrol, Pursue, Attack y RunAway

En cada método _Update()_ se añade la deteción de pulsar el botón izquierdo del ratón.

```csharp
public override void Update()
{
    // ... Resto de código ...

    // Cuando el jugador hace clic con el botón izquierdo del ratón, se instancia la calse Die
    if (Input.GetMouseButton(0))
    {
        nextState = new Die(npc, agent, animator, player);
        stage = EVENT.EXIT;
    }
}
```

### Clase Die

```csharp
public class Die : State
{
    float t = 0;
    bool showMsg;

    public Die(GameObject _npc, NavMeshAgent _agent, Animator _animator, Transform _player)
        : base(_npc, _agent, _animator, _player)
    {
        name = STATE.DIE;
    }

    // Cuando entra, se activa la animación y se para el movimiento del NPC
    public override void Enter()
    {
        animator.SetTrigger("isDie");
        agent.isStopped = true;
        base.Enter();
    }

    // Mensaje informativo de que el NPC está muerto
    public override void Update()
    {
        
        float delay = 3f;
        

        t += Time.deltaTime;
        if (t > delay)
        {
            if (!showMsg)
            {
                Debug.Log("NPC muerto");
                showMsg = true;
            }
        }
    }

    public override void Exit()
    {  
        base.Exit();
    }
}
```

Para la animación, he modificado el _Animator_.

![Animator](animator.png)

He añadido la animación **Dying** (Dying 0) y he creado una transición desde _Any State_ con un _Trigger_ llamado _isDie_.

> Cuando hace la animación, el NPC queda suspendido en el aire, no lo he modificado ya que no era parte del ejercicio.

## 09. Imprimir el estado del NPC

Para que se tener conocimiento en que estado está el NPC, he modificado el código del script _AI_.

```csharp
// Propiedad
string npcState;

void Update() {
    currentState = currentState.Process();

     // Comprueba si el valor de la propiedad npcState es distinto al nombre del estado del NPC (currentState)
    if (npcState != currentState.ToString())
    {
        // Si es distinto el valor:
        //   - Guarda el valor del estado del NPC en la propiedad npcState
        //   - Imprime en la consola un mensaje con el nuevo estado
        npcState = currentState.ToString();
        Debug.Log("Nuevo estado: " + npcState);
    }
}
```
