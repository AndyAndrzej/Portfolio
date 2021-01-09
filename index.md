# My Portfolio
### Moje projekty
##### W Unity/C#

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
    
##### W pythonie
Do pokazania na razie brak
##### C++
1. Równanie do liczenia miejsc zerowych funkji. 
   - Zczytuję funkcję z jednego stringa, objektowe. Wymaga rozbudowania (nie uwwzględnia wszystkich możliwości)
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/r%C3%B3wnanie)
   
2. Generator wykresu 
   - Po wpisaniu funkcji generuje wykres funkcji (mało dokładnie, nie uwzględnia wielu elementów).
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/wykres)
   
3. Szachy
   - Szachy w c++ z pomocą SFML. Jeden z mych pierwszych projektów, cały strukturalny
   - [GitLab](https://gitlab.com/andrzejszablewski13/console-programs/-/tree/master/szachy)
