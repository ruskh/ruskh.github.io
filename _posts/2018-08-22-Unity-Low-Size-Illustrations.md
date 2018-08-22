---
layout: post
title: "Low Size Illustrations using SpriteShape"
date: 2018-08-22
---

Join points to create simple low size illustrations




{% highlight C# %} 
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.U2D;



[ExecuteInEditMode]  

public class ShapeGen : MonoBehaviour
{

    SpriteRenderer sourceSpriteRenderer;
    Sprite sourceSprite;
    
    PointComparer pC = new PointComparer(); //Custom Comparer  to sort points, inherits from Comparer<T>
    Vector2[] sourceVertices;
    Vector3[] sortedSourceVertices;
    float yOfstartPoint;
    Vector3 Endpoint;

    GameObject targetSprite;
    SpriteShapeController targetSpriteShapeController;
    Spline targetSpline;


    
    void Start()
    {
        // Get,sort and convert Vector2 Array of Source Image Vertices to Vector3 Array

        sourceSpriteRenderer = GetComponent<SpriteRenderer>();
        sourceSprite = sourceSpriteRenderer.sprite;
        sourceVertices = sourceSprite.vertices;
              
        Array.Sort(sourceVertices, pC);
        sortedSourceVertices = ConverttoVector3(sourceVertices);
        yOfstartPoint = sortedSourceVertices[0].y;
               

        // Find the Target Object and its components

        targetSprite = GameObject.Find("target");
        targetSpriteShapeController = targetSprite.GetComponent<SpriteShapeController>();
        targetSpline = targetSpriteShapeController.spline;
        
       
        //Insert points on Target Object

        for (int i = 0; i < sortedSourceVertices.Length; i++)
        {
            if ( sortedSourceVertices[i].y < yOfstartPoint)
            {
                InsertValidPoint(i,targetSpline, sortedSourceVertices);

            }

        }

        for (int i = sortedSourceVertices.Length - 1; i > -1; i--)
        {
            if ( sortedSourceVertices[i].y > yOfstartPoint)
            {
                InsertValidPoint(i, targetSpline, sortedSourceVertices);
                
            }

        }

        //Remove the first four points and set the new first two points
        
        targetSpline.RemovePointAt(0);
        targetSpline.RemovePointAt(1);
        targetSpline.RemovePointAt(2);
        targetSpline.RemovePointAt(3);
        
        Endpoint = new Vector3 (sortedSourceVertices[0].x , sortedSourceVertices[0].y+0.04f, 0f);
        targetSpline.SetPosition(0, sortedSourceVertices[0]);
        targetSpline.SetPosition(1, Endpoint);

    }
     // Convert Vector2 Array to Vector3 Array 
        Vector3[] ConverttoVector3 (Vector2[] SourceVector2)
        {
            Vector3[] resultingVector3 = new Vector3[SourceVector2.Length];
            for (int i = 0; i < SourceVector2.Length; i++)
            {
                Vector2 tempV2 = SourceVector2[i];
            resultingVector3[i] = new Vector3(tempV2.x, tempV2.y, 0f);
            }
            return resultingVector3;
        
        }


    // Only Insert Point if its not too close to the previous one
    public void InsertValidPoint(int counter, Spline spline, Vector3[] sourceVector)
    {
        Vector3 diff = spline.GetPosition(spline.GetPointCount() - 1) - sourceVector[counter];
        
        if (diff.magnitude > 0.04f)
        {
            spline.InsertPointAt(spline.GetPointCount(), sourceVector[counter]);

            
            spline.SetTangentMode(spline.GetPointCount()-1, ShapeTangentMode.Linear);                      
            spline.SetBevelCutoff(spline.GetPointCount()-1, 150.0f);
            spline.SetBevelSize(spline.GetPointCount()-1, 0.25f);
            spline.SetHeight(spline.GetPointCount()-1, 1.0f); 

           
        }

    }




}

{% endhighlight %}
