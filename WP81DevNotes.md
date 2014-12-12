###在Windows phone 8.1中使用页面切换动画

WP 8.1默认的页面切换动画为CommonNavigationTransitionInfo, 除此之外WP App中还可以加入其他页面切换动画，下面讲解ContinuumNavigationTransitionInfo的使用。ContinuumNavigationTransitionInfo的原意是内容上的衔接，像是产品的列表页与详细页之间的关系。产品页中列出产品的名称，点击产品的名称后跳转到该产品的详细信息页面，同时的动画是产品名称从产品也跳到详细页。有一个内容上的衔接，所有，产品页有一个元素是跳出动画，该元素在详细页是跳入动画，这里的该元素是产品名称。

使用页面切换动画，首先确保在``App.xaml.cs``中的``RootFrame_FirstNavigated()``这个事件中有这么一行代码：

	var rootFrame= sender as Frame;
	rootFrame.ContentTransitions = this.transitions ?? new TransitionCollection() { new NavigationThemeTransition() };
	rootFrame.Navigated -= this.RootFrame_FirstNavigated;

>注意：这个位置还有一个功能，如果想设置默认的系统跳转背景色，也是在这里设置，加入如下代码即可：

> 	rootFrame.Background = new SolidColorBrush(Windows.UI.Color.FromArgb(255, 114, 142, 117));

在产品列表页，产品名称这个控件可能是``Textblock``，加入跳出动画：

	ContinuumNavigationTransitionInfo.IsExitElement="True"
在详细内容页，产品名称这个控件上加入跳入动画：

	ContinuumNavigationTransitionInfo.IsEntranceElement="True"

修改详细页的``Page.Transitions`` 的``NavigationThemeTransition``，将默认动画修改为``ContinuumNavigationTransitionInfo``:

	<Page.Transitions>
        <TransitionCollection>
            <NavigationThemeTransition>
                <NavigationThemeTransition.DefaultNavigationTransitionInfo>
                    <ContinuumNavigationTransitionInfo />
                </NavigationThemeTransition.DefaultNavigationTransitionInfo>
            </NavigationThemeTransition>           
        </TransitionCollection>
    </Page.Transitions>

###在Pivot中使用动画
PivotItem中也可以使用动画，左右滑动PivotItem的时候，可以设置内容项的加载速率，这里是通过组来实现，1组最先加载，接下来是2组的项目，继而是3组的项目。在PivotItem中可能是 Gridview 或 ListView，DataTemplate中可能是Textblock，在Textblock中设置加载组：

	Pivot.SlideInAnimationGroup="GroupOne"
	Pivot.SlideInAnimationGroup="GroupTwo"

###使用GridView 
GridView 是一个数据容器，它以行+列的格式显示数据，如果想改变一行中显示的数据个数，可以使用VariableSizedWrapGrid中的MaximumRowsOrColumns:

	<VariableSizedWrapGrid Orientation="Horizontal" MaximumRowsOrColumns="1" />

是一个纵向的列表，每行显示一个。

	<VariableSizedWrapGrid Orientation="Vertical" MaximumRowsOrColumns="1" />
是一个横向的列表，每列显示一个。

Gridview展示：

    <GridView x:Name="gv" Header="GrideView Display" HorizontalContentAlignment="Left" Foreground="{ThemeResource PhoneAccentBrush}" FontSize="21" SelectionChanged="listBox_SelectionChanged">
        <GridView.ItemTemplate>
            <DataTemplate>
                <StackPanel Orientation="Vertical" Margin="0,2.5">
                    <TextBlock Text="{Binding Title}"  Style="{ThemeResource ListViewItemTextBlockStyle}" 
                               HorizontalAlignment="Left" Width="400" 
                               Foreground="{ThemeResource ApplicationForegroundThemeBrush}" 
                               Pivot.SlideInAnimationGroup="GroupOne" 
                               ContinuumNavigationTransitionInfo.IsExitElement="True"/>
                    <TextBlock Text="{Binding Discription}" Style="{ThemeResource ListViewItemContentTextBlockStyle}" Foreground="{ThemeResource PhoneAccentBrush}" Pivot.SlideInAnimationGroup="GroupTwo" />
                </StackPanel>
            </DataTemplate>
        </GridView.ItemTemplate>
        <GridView.ItemsPanel>
            <ItemsPanelTemplate>
                <VariableSizedWrapGrid Orientation="Horizontal" MaximumRowsOrColumns="1" />
            </ItemsPanelTemplate>
        </GridView.ItemsPanel>
    </GridView>

GridView 的SelectionChanged事件：

 	GridView lb = sender as GridView;
    if (lb!=null && lb.SelectedIndex != -1)
    {
        string name = ((Book)lb.SelectedValue).Title.ToString();
	}


###使用SemanticZoom控件
SemanticZoom的source是一组有分类的列表，最常见的分类是按照字母顺序。SemanticZoom有两个View，其一是``SemanticZoom.ZoomedOutView``，其二是``SemanticZoom.ZoomedInView``。ZoomOutView显示分组名称，ZoomInView显示详细列表。

首先，准备数据源Source，在页面加入静态Resources，如下：

	<Page.Resources>
        <CollectionViewSource x:Name="myCVS" IsSourceGrouped="true" />
    </Page.Resources>

其次，在后台给Resources绑定数据，这个数据需要有分组的数据。

我的数据格式如下：

	class Book
    {
        public string Title { get; set; }
        public string Key { get; set; }
    }

	List<Book> sources = new List<Book>
	{
		new Book(){Title="计算机组成原理", Key="J"},
		new Book(){Title="大学英语", Key="D"},
		new Book(){Title="大学数学", Key="D"},
		new Book(){Title="计算机英语", Key="J"},
	};

为myCVS绑定数据：

	myCVS.Source = from g in sources group g by g.Key into grp orderby grp.Key select grp;
    (semanticZoom.ZoomedOutView as ListViewBase).ItemsSource = myCVS.View.CollectionGroups;

最后，在页面上为ZoomedInView绑定数据：

	<ListView x:Name="lvAZ" ItemsSource="{Binding Source={StaticResource myCVS}}" SelectionChanged="ListView_SelectionChanged"  ></ListView>

整体上SentanticZoom如下图：

    <SemanticZoom x:Name="semanticZoom">
        <SemanticZoom.ZoomedOutView>
            <GridView>
                <GridView.ItemTemplate>
                    <DataTemplate>
                        <Border BorderThickness="2.5" Background="{ThemeResource PhoneAccentBrush}" Width="65" Height="65">
                            <TextBlock
                    Text="{Binding Group.Key}"
                    FontFamily="Segoe UI" FontWeight="Light"
                    FontSize="24" HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </Border>
                    </DataTemplate>
                </GridView.ItemTemplate>
                <GridView.ItemContainerStyle>
                    <Style TargetType="GridViewItem">
                        <Setter Property="Margin" Value="4" />
                        <Setter Property="Padding" Value="10" />
                        <Setter Property="BorderBrush" Value="Gray" />
                        <Setter Property="BorderThickness" Value="1" />
                        <Setter Property="VerticalAlignment" Value="Center"/>
                        <Setter Property="HorizontalContentAlignment" Value="Center" />
                        <Setter Property="VerticalContentAlignment" Value="Center" />
                    </Style>
                </GridView.ItemContainerStyle>
            </GridView>
        </SemanticZoom.ZoomedOutView>
        <SemanticZoom.ZoomedInView>
            <ListView x:Name="lvAZ" ItemsSource="{Binding Source={StaticResource myCVS}}" SelectionChanged="ListView_SelectionChanged"  >
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <StackPanel Margin="0,0,0,24">
                            <TextBlock
                        Text="{Binding Title}"
                        TextWrapping="WrapWholeWords"
                        Pivot.SlideInAnimationGroup="1"
                        ContinuumNavigationTransitionInfo.IsExitElement="True"
                        Style="{ThemeResource ListViewItemTextBlockStyle}"/>
                        </StackPanel>
                    </DataTemplate>
                </ListView.ItemTemplate>
                <ListView.GroupStyle>
                    <GroupStyle>
                        <GroupStyle.HeaderTemplate>
                            <DataTemplate>
                                <Border BorderThickness="1" BorderBrush="{ThemeResource PhoneAccentBrush}" Width="50" Height="50">
                                    <TextBlock Text='{Binding Key}' Foreground="{ThemeResource PhoneAccentBrush}" Margin="5" FontSize="36" FontFamily="Segoe UI" FontWeight="Light" />
                                </Border>
                            </DataTemplate>
                        </GroupStyle.HeaderTemplate>
                    </GroupStyle>
                </ListView.GroupStyle>
            </ListView>
        </SemanticZoom.ZoomedInView>
    </SemanticZoom>