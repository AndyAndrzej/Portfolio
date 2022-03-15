# Moje Portfolio
### Dane Kontaktowe
Imie i nazwisko: Andrzej Szablewski

e-mail: andrzejszablewski13@gmail.com

Obecnie student 2 roku na Collegium Da Vinci Na kierunku Inforamtyka Spec. Projektowaniu Gier
### Moje projekty
#### W Unity/C#

1. Detekcja nierównego terenu i korekcja poruszania na rigidbody ( z użyciem Zenjecta i Unity.Jobs)
```c#

using System.Collections;
using System.Collections.Generic;
using Unity.Collections;
using Unity.Jobs;
using UnityEngine;
using Zenject;
public class SlopeDetector : MonoBehaviour,ICalcPhysics
{
    [SerializeField] private bool _on = true;
    [SerializeField] private float _detectRange = 5f;
    public bool On => _on;
    private Rigidbody _rb;
    private CharacterData _data;
    private IGroundDet _groundDet;
    private NativeArray<RaycastHit> _results;
    private NativeArray<RaycastCommand> _commands;
    private JobHandle _handle;
    private RaycastHit _batchedHit;
    [Inject]
    private void Construct(Rigidbody rb, CharacterData data, IGroundDet groundDet)
    {
        _rb = rb;
        _data = data;
        _groundDet = groundDet;
    }
    void Start()
    {
        _results = new NativeArray<RaycastHit>(1, Allocator.Persistent);
        _commands = new NativeArray<RaycastCommand>(1, Allocator.Persistent);
    }
    public void OnFixedUpdate()
    {
        if(_on && !_rb.isKinematic && _groundDet.IsGrounded)
        {
            _handle.Complete();
            _batchedHit = _results[0];

            if (_batchedHit.collider != null)
            {
                if(_batchedHit.normal!=Vector3.up)
                {
                    _data.SetSlopeDet(true, _batchedHit);
                }else
                {
                    _data.SetSlopeDet();
                }
            }


            _commands[0] = new RaycastCommand(_rb.transform.position,
               Vector3.down * _detectRange,
               _detectRange,int.MaxValue);
            _handle = RaycastCommand.ScheduleBatch(_commands, _results, 1);
        }
    }
    private void OnDestroy()
    {
        _handle.Complete();
        _results.Dispose();
        _commands.Dispose();
    }
}
```

2.Kod GameMenagera z obsługą multika na photonie (fragment starego projektu TRacer)
[GitLab](https://gitlab.com/andrzejszablewski13/gra/-/blob/master/Giereczka/Assets/Scripts/MultiRoom/GameManager.cs)
```c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;
using UnityEngine.SceneManagement;
using UnityEngine.UI;
using TMPro;


// Start is called before the first frame update
namespace MultiSystem
{
    public class GameManager : MonoBehaviourPunCallbacks
    {
        public enum StringSave//string save
        {
            Control, Car, Player,
            SendVoteForStart, AddMyCar, StartGame, DestroyCar, SendNameofPickUp, AfterColectingPickUp, AfterInstantializePickUp, SendNameofPlayer,
            Horizontal, OnMyDeath, Vertical, CamRig, CamPosition, PickUp
        }
        public enum NamesOfPickUps
        {
            Speed, Jump, Eraser, PickUp, JumpOnAll, SpeedUpOnAll
        }
        public Dictionary<string, Dictionary<string, GameObject>> DictionaryOFPlayersObjects = new Dictionary<string, Dictionary<string, GameObject>>();//dic for car and control comunication
        public Transform Cars, Controlers;
        public int _playerCount;
        public bool Gamepaused = true;
        public static GameManager InstancePlace { get; private set; }
        private int _voting = 0, _losing = 0;
        private PhotonView _photonView;
        [SerializeField] private Button _buttonForStart;
        [SerializeField] private TextMeshProUGUI _text;
        public Button buttonForPickUp;
        [SerializeField] private GameObject _buttonsHandHeld;
        private string[] _textOnTheEnd = new string[3] { "You Lose", "You Win/Lose against Yourself", "You Win" };
        [HideInInspector] public GameObject CanvasGameObject;
        [HideInInspector] public List<string> _listOfVotedPlayer = new List<string>();

        private void Awake()
        {
            _text.gameObject.SetActive(false);
            _photonView = this.GetComponent<PhotonView>();
            InstancePlace = this;
            PhotonNetwork.MinimalTimeScaleToDispatchInFixedUpdate = 1;
            Time.timeScale = 0;
            _buttonForStart.onClick.AddListener(AddVoteForStart);
            Gamepaused = true;
            if (SystemInfo.deviceType != DeviceType.Handheld)//seting control system handheld/PC
            {
                _buttonsHandHeld.SetActive(false);
            }
            else
            {
                _buttonsHandHeld.SetActive(true);
            }
        }
        private void AddVoteForStart()//vote for game start
        {
            _buttonForStart.interactable = false;
            _photonView.RPC(StringSave.SendVoteForStart.ToString(), RpcTarget.AllBuffered, PhotonNetwork.LocalPlayer.NickName);
        }
        public override void OnLeftRoom()
        {
            PhotonNetwork.Disconnect();
            SceneManager.LoadScene(0);

        }
        public void OnDeath()
        {
            _photonView.RPC(StringSave.OnMyDeath.ToString(), RpcTarget.AllBuffered);
            _text.gameObject.SetActive(true);
            if (_losing < _voting)
            {
                _text.text = _textOnTheEnd[0];
            }
            else if (_losing == _voting && _voting == 1)
            {
                _text.text = _textOnTheEnd[1];
            }
            else
            {
                _text.text = _textOnTheEnd[2];
            }

        }
        [PunRPC]
        private void OnMyDeath()
        {
            _losing++;
        }
        public void LeaveRoom()
        {
            PhotonNetwork.LeaveRoom();
        }
        [PunRPC]
        private void SendVoteForStart(string _nick)
        {
            _listOfVotedPlayer.Add(_nick);
            _voting++;
            if (PhotonNetwork.IsMasterClient && _voting == PhotonNetwork.CurrentRoom.PlayerCount)
            {
                _photonView.RPC(StringSave.StartGame.ToString(), RpcTarget.AllBuffered);
            }
        }
        [PunRPC]
        private void StartGame()//start game
        {
            _buttonForStart.gameObject.SetActive(false);
            Time.timeScale = 1;
            PhotonNetwork.MinimalTimeScaleToDispatchInFixedUpdate = -1;
            Gamepaused = false;
        }
    }
}
```
3. Customowy Renderer kostek na DOTSie
[GitLab](https://gitlab.com/andrzejszablewski13/GLTKGameJam-Joined-Togehter/-/blob/feature_analitics/Assets/_Game/Dots/Scripts/Systems/RenderSyStem.cs)
```c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.Entities;
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using Unity.Transforms;
using Unity.Mathematics;
using UnityEngine.Rendering;
using Unity.Collections.LowLevel.Unsafe;
[UpdateInGroup(typeof(SimulationSystemGroup),OrderLast =true)]
public class RenderSyStem : SystemBase
{
    private Material material;

    private ComputeBuffer meshPropertiesBuffer;

    private Mesh mesh;
    private Bounds bounds;
    private int shaderPropertyId;
    private struct MeshProperties
    {
        public Matrix4x4 mat;
        public Vector4 color;

        public static int Size()
        {
            return
                sizeof(float) * 4 * 4 + // matrix;
                sizeof(float) * 4;      // color;
        }
    }
    [BurstCompile]
    private struct FillArraysParallelJob : IJobParallelFor
    {
        [ReadOnly] public NativeArray<PositionData> nativeArray;
        [NativeDisableContainerSafetyRestriction] public NativeArray<MeshProperties> matrixArray;
        [ReadOnly] public NativeArray<IDInCube> IDArray;
        [ReadOnly] public NativeArray<ColorData> colorData;

        public void Execute(int index)
        {
            float valuedigit = 2000;
            PositionData renderData = nativeArray[index];
            var tempColor =colorData[index].Value switch
            {
                ColorCode.goldGrid =>Color.Lerp(new Vector4(1,0.92f,0.016f,0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.redGrid => Color.Lerp(new Vector4(1, 0, 0, 0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.blueGrid => Color.Lerp(new Vector4(0, 0, 1, 0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.purpleGrid => Color.Lerp(new Vector4(1, 0, 1, 0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.cyanGrid => Color.Lerp(new Vector4(0, 1, 1, 0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.greenGrid => Color.Lerp(new Vector4(0, 1, 0, 0.3f), new Vector4(1, 1, 1, 0.05f), IDArray[index].ID / valuedigit),
                ColorCode.gold => Color.yellow,
                ColorCode.red => Color.red,
                ColorCode.blue => Color.blue,
                ColorCode.purple => Color.magenta,
                ColorCode.cyan => Color.cyan,
                ColorCode.green => Color.green,
                ColorCode.toDestroy => Color.black,
                _ => Color.white,
            };
            MeshProperties temp = new MeshProperties
            {
                mat = Matrix4x4.TRS(nativeArray[index].Value, Quaternion.identity, Vector3.one * 0.1f),
                color = tempColor
            };
            matrixArray[index] = temp;
        }

    }

    [BurstCompile]
    private struct EnviromentalCubes : IJobParallelFor
    {
        [ReadOnly] public NativeArray<PositionData> nativeArray;
        [NativeDisableContainerSafetyRestriction] public NativeArray<MeshProperties> matrixArray;
        [ReadOnly] public NativeArray<FullColorData> Selcolor;
        [ReadOnly] public int startindex;
        public void Execute(int index)
        {
            MeshProperties temp = new MeshProperties
            {
                mat = Matrix4x4.TRS(nativeArray[index].Value, Quaternion.identity, Vector3.one),
                color = Selcolor[index].value
            };
            matrixArray[index+ startindex] = temp;
        }

    }
    private EntityQueryDesc _querryAsk, _querryAskEnv;
    protected override void OnCreate()
    {
        base.OnCreate();
        EntityQueryDesc startQueryAsk = new EntityQueryDesc()
        {
            All = new ComponentType[]
{
            ComponentType.ReadOnly<PositionData>(),
}
        };
        RequireForUpdate(GetEntityQuery(startQueryAsk));
        bounds = new Bounds(Vector3.zero, Vector3.one * 10);
        shaderPropertyId = Shader.PropertyToID("_Properties");
        _querryAsk = new EntityQueryDesc
        {
            All = new ComponentType[] { ComponentType.ReadOnly<PositionData>(), ComponentType.ReadOnly<IDInCube>(), ComponentType.ReadOnly<ColorData>() }
        };
        _querryAskEnv = new EntityQueryDesc
        {
            All = new ComponentType[] { ComponentType.ReadOnly<PositionData>(), ComponentType.ReadOnly<EnvTag>(), ComponentType.ReadOnly<FullColorData>() }
        };
    }
    MaterialPropertyBlock proBlock;
    protected override void OnUpdate()
    {
        if (GameHandler.GetInstance() != null && GameHandler.GetInstance().mapData!=null)
        {
            mesh = GameHandler.GetInstance().quadMesh;
            material = GameHandler.GetInstance().Mat;
            if (meshPropertiesBuffer != null)
            {
                meshPropertiesBuffer.Release();
            }
            meshPropertiesBuffer = null;

            EntityQuery entityQuery = GetEntityQuery(_querryAsk);
            EntityQuery entityQueryEnv = GetEntityQuery(_querryAskEnv);
            NativeArray<PositionData> translationArray = entityQuery.ToComponentDataArray<PositionData>(Allocator.TempJob);
            NativeArray<PositionData> translationArray2 = entityQueryEnv.ToComponentDataArray<PositionData>(Allocator.TempJob);
            NativeArray<FullColorData> fullColorArray = entityQueryEnv.ToComponentDataArray<FullColorData>(Allocator.TempJob);
            NativeArray<IDInCube> IDArray = entityQuery.ToComponentDataArray<IDInCube>(Allocator.TempJob);
            NativeArray<ColorData> colordata = entityQuery.ToComponentDataArray<ColorData>(Allocator.TempJob);
            NativeArray<MeshProperties> propnArray = new NativeArray<MeshProperties>(translationArray.Length+translationArray2.Length, Allocator.TempJob);
            FillArraysParallelJob fillArraysParallelJob = new FillArraysParallelJob
            {
                nativeArray = translationArray,
                matrixArray = propnArray,
                IDArray = IDArray,
                colorData= colordata
            };
            EnviromentalCubes EnvCubes = new EnviromentalCubes
            {
                nativeArray = translationArray2,
                matrixArray = propnArray,
                startindex = translationArray.Length,
                Selcolor = fullColorArray
            };
            EnvCubes.Schedule(translationArray2.Length, 10).Complete();
            fillArraysParallelJob.Schedule(translationArray.Length, 10).Complete();
            meshPropertiesBuffer = new ComputeBuffer(propnArray.Length, MeshProperties.Size());
            meshPropertiesBuffer.SetData(propnArray);
            //material.SetBuffer(shaderPropertyId, meshPropertiesBuffer);
            proBlock = new MaterialPropertyBlock();
            proBlock.SetBuffer(shaderPropertyId, meshPropertiesBuffer);
            Graphics.DrawMeshInstancedProcedural(mesh, 0, material, bounds, propnArray.Length, proBlock, ShadowCastingMode.On,true);

            translationArray.Dispose();
            propnArray.Dispose();
            IDArray.Dispose();
            colordata.Dispose();
            translationArray2.Dispose();
            fullColorArray.Dispose();
        }
    }
    protected override void OnDestroy()
    {
        if (meshPropertiesBuffer != null)
        {
            meshPropertiesBuffer.Release();
        }
        meshPropertiesBuffer = null;
    }
}
```
4. Line renderer na canvas (unity)
   - Line renderer który działa na canvasie, zawiera kilka podsatawowych opcji, krótki i w miarę wydajny kod
   - [GitLab](https://gitlab.com/andrzejszablewski13/doomwave/-/blob/master/DoomW/DoomW/Assets/Scripts/Canvas/LineRendererHUD.cs)
   - Kod:
   
    ```c#
    
    ﻿using System.Collections.Generic;
    using UnityEngine;
    using UnityEngine.UI;

    //to draw a line on UI from the attached Object
    public class LineRendererHUD : Graphic
    {
    public float thickness;
    private float _refthickness;

    public List<Vector2> points;

    private float _width;
    private float _height;
    private float _angle;
    private Vector2 _point, _point2;
    private int _numberOfPoints=4;
    private UIVertex _vertex;
    protected override void OnPopulateMesh(VertexHelper vh)
    {
        vh.Clear();
        _refthickness = thickness;
        _width = this.transform.localPosition.x;
        _height = this.transform.localPosition.y;//start pos as attached gameobject
        if (points.Count < 2) return;
        _angle = 0;
        for (int i = 0; i < points.Count - 1; i++)
        {
            _point = points[i];
            _point2 = points[i + 1];//points are middle points of the line
            if (i < points.Count - 1)
            {
                _angle = GetAngle(points[i], points[i + 1]) + 90f;
            }
            _vertex = UIVertex.simpleVert;
            _vertex.color = color;
            AddVertex(_angle, ref _point, vh, ref _vertex);
            AddVertex(_angle, ref _point, vh, ref _vertex);//two points -one point for each side of middle point of line
            AddVertex(_angle, ref _point2, vh, ref _vertex);
            AddVertex(_angle, ref _point2, vh, ref _vertex);//next two points -one point for each side of middle point of line-together 4 for one rectangle of line
        }
        for (int i = 0; i < points.Count - 1; i++)
        {
            vh.AddTriangle(i* _numberOfPoints + 0, i * 4 + 1, i * _numberOfPoints + 2);
            vh.AddTriangle(i * _numberOfPoints + 1, i * _numberOfPoints + 2, i * _numberOfPoints + 3);
        }
    }
    public float GetAngle(Vector2 me, Vector2 target)
    {
        //panel resolution go there in place of 9 and 16
        return (float)(Mathf.Atan2(9f * (target.y - me.y), 16f * (target.x - me.x)) * (180 / Mathf.PI));
    }
    private void AddVertex(float angle,ref Vector2 point,VertexHelper vh,ref UIVertex vertex)
    {
        _refthickness *= -1;//to rotate to another side
        vertex.position = Quaternion.Euler(0, 0,angle) * new Vector3(_refthickness / 2, 0);
        vertex.position += new Vector3(point.x + _width, point.y + _height);
        vh.AddVert(vertex);
    }
    }

    ```
5. The Title of the game/Those are Cubes
   - gra z gamejama (GLTK 2021), gra logiczna, rozwinięta na mobilkę i połączona z DOTS
   - [GitLab](https://gitlab.com/andrzejszablewski13/GLTKGameJam-Joined-Togehter)
   - [itch.io](https://necrogames.itch.io/the-title-of-the-game)
6. 3D Trail Renderer
   - 3D trail renderer, polegający na edycji meschy, generuje trail który interpretuje kolizje, kod w miarę efektywny (przemyślany na mobilki), kod bez komentarzy
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra/-/blob/master/Giereczka/Assets/Scripts/Car/Trail3D.cs)
7. AI On State
   -nieudane ai na stetach (złe podejście do problemu- sterowaniem pojazdu na wheel colitherach) ale zrobione zgodnie z definicją
   -[GitLab](https://gitlab.com/andrzejszablewski13/gra/-/tree/master/Giereczka/Assets/Scripts/AIOnState)
8. Lobby na photonie
   - lobby do gry na photonie, pokoje mozliwe do tworzenia, maks 4 gracze, z opcja tworzenia nicku i wyboru postaci, kod bez komentarzy
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra/-/tree/master/Giereczka/Assets/Scripts/Lobby)
9. ML-agents
   - moje próby z użyciem uczenia maszynowego
   - [GitLab](https://gitlab.com/andrzejszablewski13/my-own-mlagents-try/-/tree/master/Project/Assets/ML-Agents/OwnTries)
   - projekt prowadzi do kilku zapisanych folderów z projektami
      * RollerBall to jest startowy tutorialowy kod z tutoriali twórców.
      * RLWitchCars to jest mój pierwszy włąsny projekt z połaczeniem pojazdów na wheel colitherach. AI w miarę działa ale nie dodałem detekcji czy reagowanie na wyjazd poza                platformę a tylko dojazd do celu (losowo wybranego kwadratu)
      * Kółko i krzyżyk to kolejny w maire udany projekt ai do gry w kółko i krzyzyk. Jest ono w miare działajace (choc czasem lubi sie popsuc i wymagac ponownego przetrenowania            na tym samym kodzie), kod niezaladny i posiadajacy wiele miejsc do usprawnien (zwlaszcza od strony mastera do gry)
      * Reversi ostatni projekt nieudany z powodu zbyt duzego rozmiaru w porówaniu do posiadanego sprzetu
      * config to folder z plikami konfiguracyjnymi (.yaml) i gotowymi wytrenowanymi AI

10. Pierwsza (w miare skonczona i mozliwa do pokazania) gra TRacer
   - gra o pojazdach i zabijaniu przeciwników własnym trailem (jak w tronie)
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra)

11. TRacerV2
   - przerobiony TRacer, bez multi ale z botami, w formie wyścigu
   - [GitLab](https://gitlab.com/andrzejszablewski13/TRacerV2)

    
#### W pythonie
1. Program do wyliczania pierwiastka funkcji
   - program wyliczający pierwiastek funkcji, funkcja podana ze stringa,ograniczona liczba funkcji które można odczytać
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/PythonPrograms)
2. TensorFlow
   - moje próby z tensorflow, zwiera kilka podsatawowych tutoriali i moja próba stworzenia zewnętrznego bota do gry bulletHell (nieudana gdyż czcionka scora i health byłą wysoce       niestabilna do zewnętrznego odczytania). Kod słabo opisany ale zawiera wiele zaawansowanych elementów jak odczytywanie ekranu, objektowość i wielowątkowość
   - [GitLab](https://gitlab.com/andrzejszablewski13/phytonscripts2)
   

#### C++
1. Funkcja do wyliczania y z równania/funkcji. 
   - Zczytuję funkcję z jednego stringa, objektowe. Wymaga rozbudowania (nie uwwzględnia wszystkich możliwości)
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/r%C3%B3wnanie)
   
2. Generator wykresu 
   - Po wpisaniu funkcji generuje wykres funkcji (mało dokładnie, nie uwzględnia wielu elementów), czytanie równania z powyższego.
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/wykres)
   
3. Szachy
   - Szachy w c++ z pomocą SFML. Jeden z mych pierwszych projektów, cały strukturalny
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/szachy)
