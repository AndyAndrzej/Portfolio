# Moje Portfolio
### Dane Kontaktowe
Imie i nazwisko: Andrzej Szablewski

e-mail: andrzejszablewski13@gmail.com

Obecnie student 2 roku na Collegium Da Vinci Na kierunku Inforamtyka Spec. Projektowaniu Gier
### Moje projekty
#### W Unity/C#

1. Line renderer na canvas (unity)
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
2. Moving Lens Than Camera
   - scrypt do sunchronizacjo kamer służących np. za lustro bądz portal z kamerą gracza, operuje na fizycznych wlasnosciach kamery, mniej na jej poruszaniu,       (nieco zabagowana przy małych odległościach) ale za to bez problemu przycina widok tuż przed lustrem/portalem niezależnie od kątu patrzenia
   - [GitLab](https://gitlab.com/andrzejszablewski13/portal-prototype/-/blob/master/PortalExperimental/Assets/ElementsForPortalsBase/MovingLensThanFerment.cs)
   - Kod (z komentarzami):
   
   ```c#
   using System.Collections;
   using System.Collections.Generic;
   using UnityEngine;
   using UnityEngine.UIElements;

   namespace Portal
   {
    public class MovingLensThanFerment : MonoBehaviour
    {
        private GameObject _portal, _mainCamera;

        private Camera _cameraPortal, _targetCamera;
        private GameObject _planePortal,_pseudoMainCamera;
        private float _frustumHeight, _distance, _frustumWidth,_lensWidth,_lensHeight;
        private GameObject _target,_targetPseudoCamera;
        private float _distanceZ, _distanceX, _distanceY;
        private float _correctPrecision = 0.1f;//for camera render just above plane of mirror/portal

        // Start is called before the first frame update

        private void Start()
        {
            _portal = this.gameObject;//portal//mirror main object
            _target = _portal == PortalSingleton.Singleton.Portal1 ? _target = PortalSingleton.Singleton.Portal2 : _target = PortalSingleton.Singleton.Portal1;//target that will render img to show (texture is added manually)
            //change above if for mirror for the same object as _portal
            _targetCamera = _target.GetComponentInChildren<Camera>();
            _targetPseudoCamera= _target.GetComponentInChildren<PseudoMainCamera>().gameObject;//psuedoMainCamera class is empty class
            _mainCamera = PortalSingleton.Singleton.mainCamera;
            _cameraPortal = _portal.transform.GetComponentInChildren<Camera>();//camera to render image from this mirror/portal
            _planePortal= _portal.transform.GetChild(0).gameObject;//plane of mirror/portal
            _pseudoMainCamera= _portal.transform.GetComponentInChildren<PseudoMainCamera>().gameObject;//pos of camera if was moving not by lens
        }
        // mova camera by lens than by pos change (only x and y axis)
        void FixedUpdate()
        {
            SetCameraPosition(_targetCamera, _portal, _targetPseudoCamera, _mainCamera);
            _distance = Vector3.Distance(_cameraPortal.transform.position, _planePortal.transform.position);
            _frustumHeight = 2.0f * _distance * Mathf.Tan(_cameraPortal.fieldOfView * 0.5f * Mathf.Deg2Rad);
            _frustumWidth = _frustumHeight * _cameraPortal.aspect;
            _lensWidth = (_planePortal.transform.position.x - _pseudoMainCamera.transform.position.x) / _frustumWidth;
            _lensHeight = (_planePortal.transform.position.y - _pseudoMainCamera.transform.position.y) / _frustumHeight;
            _cameraPortal.lensShift = new Vector2(_lensWidth, _lensHeight);
        }
        //move camera on z axis and pseudo camera rest paramets
        private void SetCameraPosition(Camera _cameraToSet, GameObject _portal, GameObject _pseudoMainCamera,GameObject _mainCamera)
        {
            _distanceX = _mainCamera.transform.position.x - _portal.transform.position.x;
            _distanceY = _mainCamera.transform.position.y - _portal.transform.position.y;
            _distanceZ = _mainCamera.transform.position.z - _portal.transform.position.z;
            _cameraToSet.transform.localPosition = new Vector3(_cameraToSet.transform.localPosition.x, _cameraToSet.transform.localPosition.y, _distanceZ);
            _pseudoMainCamera.transform.localPosition = new Vector3(_distanceX, _distanceY, _pseudoMainCamera.transform.localPosition.z);
            _pseudoMainCamera.transform.localEulerAngles = _mainCamera.transform.eulerAngles;
            _cameraToSet.nearClipPlane = Mathf.Abs(_distanceZ) + _correctPrecision;
        }
    }
   }
   ```

3. 3D Trail Renderer
   - 3D trail renderer, polegający na edycji meschy, generuje trail który interpretuje kolizje, kod w miarę efektywny (przemyślany na mobilki), kod bez komentarzy
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra/-/blob/master/Giereczka/Assets/Scripts/Car/Trail3D.cs)
4. AI On State
   -nieudane ai na stetach (złe podejście do problemu- sterowaniem pojazdu na wheel colitherach) ale zrobione zgodnie z definicją
   -[GitLab](https://gitlab.com/andrzejszablewski13/gra/-/tree/master/Giereczka/Assets/Scripts/AIOnState)
5. Lobby na photonie
   - lobby do gry na photonie, pokoje mozliwe do tworzenia, maks 4 gracze, z opcja tworzenia nicku i wyboru postaci, kod bez komentarzy
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra/-/tree/master/Giereczka/Assets/Scripts/Lobby)
6. ML-agents
   - moje próby z użyciem uczenia maszynowego
   - [GitLab](https://gitlab.com/andrzejszablewski13/my-own-mlagents-try/-/tree/master/Project/Assets/ML-Agents/OwnTries)
   - projekt prowadzi do kilku zapisanych folderów z projektami
      * RollerBall to jest startowy tutorialowy kod z tutoriali twórców.
      * RLWitchCars to jest mój pierwszy włąsny projekt z połaczeniem pojazdów na wheel colitherach. AI w miarę działa ale nie dodałem detekcji czy reagowanie na wyjazd poza                platformę a tylko dojazd do celu (losowo wybranego kwadratu)
      * Kółko i krzyżyk to kolejny w maire udany projekt ai do gry w kółko i krzyzyk. Jest ono w miare działajace (choc czasem lubi sie popsuc i wymagac ponownego przetrenowania            na tym samym kodzie), kod niezaladny i posiadajacy wiele miejsc do usprawnien (zwlaszcza od strony mastera do gry)
      * Reversi ostatni projekt nieudany z powodu zbyt duzego rozmiaru w porówaniu do posiadanego sprzetu
      * config to folder z plikami konfiguracyjnymi (.yaml) i gotowymi wytrenowanymi AI

7. Pierwsza (w miare skonczona i mozliwa do pokazania) gra TRacer
   - gra o pojazdach i zabijaniu przeciwników własnym trailem (jak w tronie)
   - [GitLab](https://gitlab.com/andrzejszablewski13/gra)
8. DoomWave
   - obecnie rozwijana gra, typu rogue-like+doom z opcja tworzenia wlasnych broni z modulów
   - [GitLab](https://gitlab.com/andrzejszablewski13/doomwave)

    
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
