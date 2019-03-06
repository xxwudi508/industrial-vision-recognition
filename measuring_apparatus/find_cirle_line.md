###  一键测量机halcon算法

1. **一键找直线**

   > 主要算子 : measure_pos
   >
   > 说明：左键画矩形，选择算子属性，确定ROI找直线的方向等

   示例：

   **halcon12：**

   ```python
   dev_set_draw('margin')
   read_image(img,'test.jpg')
   draw_rectangle1(3600, Row1, Column1, Row2, Column2)
   get_image_size(img, nWidth, nHeight)
   gen_rectangle1(Rectangle, Row1, Column1, Row2, Column2)
   orientation_region(Rectangle, Angle1)
   *m_direction找直线方向 ||| 0：上下，1：左右
   m_direction:=0
   *m_transition边缘极性方向 ||| positive：从白到黑，negative：从黑道白
   m_transition:='negative'
   *m_point找点的属性 ||| first:第一个点，last：最后一个点
   m_point:='first'
   *m_sum_point搜索的次数 ||| int
   m_sum_point:=30
   *m_thresh 阈值
   m_thresh:=30
   
   smallest_rectangle2(Rectangle, CenterY, CenterX, Phi, HalfWidth, HalfHeight)
   PI:=3.14159
   Pt1Y:=CenterY+sin(Phi)*HalfWidth-sin(PI/2.0-Phi)*HalfHeight
   Pt1X:=CenterX-cos(Phi)*HalfWidth-cos(PI/2.0-Phi)*HalfHeight
   Pt2Y:=CenterY-sin(Phi)*HalfWidth-sin(PI/2.0-Phi)*HalfHeight
   Pt2X:=CenterX+cos(Phi)*HalfWidth-cos(PI/2.0-Phi)*HalfHeight
   gen_cross_contour_xld(Cross10, Pt1Y, Pt1X, 80, Phi)
   gen_cross_contour_xld(Cross11, Pt2Y, Pt2X, 80, Phi)
   
   Pt3Y:=CenterY+sin(Phi)*HalfWidth+sin(PI/2.0-Phi)*HalfHeight
   Pt3X:=CenterX-cos(Phi)*HalfWidth+cos(PI/2.0-Phi)*HalfHeight
   Pt4Y:=CenterY-sin(Phi)*HalfWidth+sin(PI/2.0-Phi)*HalfHeight
   Pt4X:=CenterX+cos(Phi)*HalfWidth+cos(PI/2.0-Phi)*HalfHeight
   gen_cross_contour_xld(Cross12, Pt3Y, Pt3X, 80, Phi)
   gen_cross_contour_xld(Cross13, Pt4Y, Pt4X, 80, Phi)
   
   gen_region_line(RegionLines, Pt1Y, Pt1X, Pt2Y, Pt2X)
   gen_region_line(RegionLines1, Pt3Y, Pt3X, Pt4Y, Pt4X)
   gen_region_line(RegionLines2, Pt1Y, Pt1X, Pt3Y, Pt3X)
   gen_region_line(RegionLines3, Pt2Y, Pt2X, Pt4Y, Pt4X)
   
   concat_obj(RegionLines2, RegionLines3, ObjectsConcat1)
   orientation_region(ObjectsConcat1, Phi1)
   tuple_deg(Phi1, Deg1)
   concat_obj(RegionLines, RegionLines1, ObjectsConcat2)
   orientation_region(ObjectsConcat2, Phi2)
   tuple_deg(Phi2, Deg2)
   
   angle:=0
   rect_angle:=0
   gen_empty_obj(SelectObject)
   if (m_direction=0)
       angle:=90
       rect_angle:=90
   else
       angle:=0
       rect_angle:=180
   endif
   
   if(angle=abs(Deg1[0]))
       SelectObject:=ObjectsConcat2
   else
       SelectObject:=ObjectsConcat1
   endif
   
   count_obj(SelectObject, Number)
       
   *第一条线的点值
   select_obj(SelectObject, ObjectSelected1, 1)
   get_region_points(ObjectSelected1, RowsLine1, ColLine1)
   *第二条线的点值
   select_obj(SelectObject, ObjectSelected1,2)
   get_region_points(ObjectSelected1, RowsLine2, ColLine2)
       
   gen_cross_contour_xld(Cross3, RowsLine1[0], ColLine1[0], 50, Angle1)
   gen_cross_contour_xld(Cross4, RowsLine2[0], ColLine2[0], 50, Angle1)
       
   ****选择间隔
   tuple_length(RowsLine1,Line1Length1)
   tuple_length(RowsLine2, Line1Length2)
   if(Line1Length1>Line1Length2)
     SelectLength:=Line1Length2
   else
      SelectLength:=Line1Length1
   endif
   **求取测量ROI的长度
   distance_pp(RowsLine1[10:SelectLength-1], ColLine1[10:SelectLength-1], RowsLine2[10:SelectLength-1], ColLine2[10:SelectLength-1], Distance2)
   Dist:=mean(Distance2)
   union1(SelectObject, RegionUnion)
   area_center(RegionUnion, Area, RowRect, ColumnRect)
       
   **匹配后的旋转角度
   
   RowCoord:=[]
   ColCoord:=[]
   iPixGap:=SelectLength/m_sum_point
       
   for i:=1 to SelectLength+1 by 1
   
       if(iPixGap * i< SelectLength)        
           CenterY:=(RowsLine1[iPixGap * i]+RowsLine2[iPixGap * i])/2.0
           CenterX:=(ColLine1[iPixGap * i]+ColLine2[iPixGap * i])/2.0
          
           gen_rectangle2(Rectangle3, CenterY, CenterX, rad(rect_angle), Dist/2.0, 5)          
           ******创建测量句柄***************
           gen_measure_rectangle2(CenterY, CenterX,rad(rect_angle),Dist/2.0,5, nWidth, nHeight, 'nearest_neighbor', MeasureHandle1)
   
           measure_pos(img, MeasureHandle1, 1, m_thresh,m_transition, m_point, RowEdge, ColumnEdge, Amplitude1, Distance3)
   
           gen_cross_contour_xld(Cross5, RowEdge, ColumnEdge, 6, Angle1)
   
           RowCoord:=[RowEdge,RowCoord]
           ColCoord:=[ColumnEdge,ColCoord]
         else
             break
             stop()
         endif
        close_measure(MeasureHandle1)
        gen_cross_contour_xld(CrossFinal, RowCoord, ColCoord, 6, 10)      
   endfor  
       
   gen_contour_polygon_xld(Contour, RowCoord, ColCoord)
   fit_line_contour_xld (Contour, 'tukey', -1, 0, 5, 2, RowBegin, ColBegin, RowEnd, ColEnd, Nr, Nc, Dist1)
   gen_contour_polygon_xld(Contour1, [RowBegin,RowEnd], [ColBegin,ColEnd])
   dev_display(img)
   dev_display(CrossFinal)
   dev_display(Contour1)
   
   ```

2. **一键找圆**

   > 主要算子 : measure_pos
   >
   > 说明：左键画矩形，选择算子属性，确定ROI找圆的方向等

   **halcon12**：

   ```python
   dev_set_draw('margin')
   read_image(img,'D:/机器视觉/一键测量机/1.jpg')
   draw_circle(3600, Row, Column, Radius)
   gen_contour_polygon_xld(Contour, Row, Column)
   get_image_size(img,nWidth,nHeight) 
   R1:=Radius-Radius
   
   gen_circle_contour_xld(ContCircle, Row, Column, Radius, 0, 6.28318, 'positive', 1)
   gen_circle_contour_xld(ContCircle1, Row, Column, R1, 0, 6.28318, 'positive', 1)
   *找圆参数
   
   *遍历个数
   m_nLinseNum:=30 
   m_dEndAngle:= 360
   m_dStartAngle:= 0
   *nCurSamplePtsNum卡尺的间隔角度
   nCurSamplePtsNum:=(360.0/(fabs(m_dEndAngle-m_dStartAngle)))*(m_nLinseNum-1) 
   
   m_dCircleCenterY:=Row
   m_dCircleCenterX:=Column
   m_dRadius:= Radius
   m2:=R1
   
   *卡尺找圆矩形的长宽
   if(R1=0)
       m_dLength:=(Radius+R1)
   else
       m_dLength:=(Radius+R1)
   endif
   m_dWidth:=10
   
   m_dSigma :=1
   m_nThreshold:=30
   strTransition:='positive'
   strSelect:='first'
   *以上为找圆参数
   
   flag:=0
   *flag = 0 表示从里面到外面， flag=1 表示从外面从里面找
   if(flag = 0 )
       strTransition:='negative'
   else
       strTransition:='positive'
   endif
   
   GenRow:=[]
   GenCol:=[]
   for Index:=0 to m_nLinseNum by 1
       
       rad1:=((2.0*3.14)/nCurSamplePtsNum)*(1.0*Index)+(m_dStartAngle/180.0*3.14)
       angel:=deg(rad1)
       tuple_sin(((2.0*3.14)/nCurSamplePtsNum)*(Index*1.0)+(m_dStartAngle/180.0*3.14), Sin)
   	tuple_cos(((2.0*3.14)/nCurSamplePtsNum)*(Index*1.0)+(m_dStartAngle/180.0*3.14), Cos)
       
       *圆弧上的点坐标
       RowRect := m_dCircleCenterY-m_dRadius*Sin
       ColumnRect:= m_dCircleCenterX+m_dRadius*Cos
       
       RowRect1 := m_dCircleCenterY-m2*Sin
       ColumnRect1:= m_dCircleCenterX+m2*Cos
   
       if(flag=0)
          tuple_atan2(-RowRect+Row, ColumnRect-Column, ATan)
       else
           tuple_atan2((-RowRect)+Row, ColumnRect-Column, ATan)
           ATan := rad(180)+ATan
       endif 
       RowL2 := RowRect+((m_dLength/2)*(sin(-ATan)))
       RowL1 := RowRect-((m_dLength/2)*(sin(-ATan)))
       ColL2 := ColumnRect+((m_dLength/2)*(cos(-ATan)))
       ColL1 :=ColumnRect-((m_dLength/2)* (cos(-ATan)))
       
       
       
       gen_cross_contour_xld(Cross1, RowRect, ColumnRect, 6, 0.785398)
       gen_arrow_contour_xld(Arrow1, RowRect, ColumnRect, RowRect1, ColumnRect1, 15, 15)
   
       gen_rectangle2(Rectangle, (RowRect+RowRect1)/2, (ColumnRect+ColumnRect1)/2, ((2.0*3.14)/nCurSamplePtsNum)*(1.0*Index)+(m_dStartAngle/180.0*3.14), m_dLength/2.0, m_dWidth/2.0)
       gen_measure_rectangle2 ((RowRect+RowRect1)/2, (ColumnRect+ColumnRect1)/2, ((2.0*3.14)/nCurSamplePtsNum)*(1.0*Index)+(m_dStartAngle/180.0*3.14), m_dLength/2.0, m_dWidth/2.0, nWidth, nHeight, 'nearest_neighbor', MeasureHandle)
      *strTransition表示极性和 strSelect要选择的点
       measure_pos(img, MeasureHandle, 1.0, m_nThreshold, strTransition, strSelect, RowEdge, ColEdge, Amplitude, Distance)
       
       gen_cross_contour_xld(Cross, RowEdge, ColEdge, 6, 0.785398)
       if(|RowEdge|>0)
           GenRow:=[GenRow,RowEdge]
           GenCol:=[GenCol,ColEdge]
       endif
       close_measure(MeasureHandle)
   endfor
   gen_cross_contour_xld(Cross2, GenRow, GenCol, 6, 0.785398)
   gen_contour_polygon_xld(Contour1, GenRow, GenCol)
   get_contour_xld(Contour1, Row3, Col1)
   *fit_circle_contour_xld用变量'geotukey'才能剔除一些偏离圆轨迹比较圆的点
   fit_circle_contour_xld (Contour1, 'geotukey', -1, 0, 0, 3, 2, Row2, Column1, Radius1, StartPhi, EndPhi, PointOrder)
   dev_set_color('red')
   gen_circle_contour_xld(ContCircle, Row2, Column1, Radius1, 0, 6.28318, 'positive', 1)
   
   ```

   