-- AquaUI - Premium Light Blue UI Library for Executors
-- Made for FPS/Shooter Games (Arsenal, Phantom Forces, Bad Business style)
-- Executor-friendly: No dependencies, no require()

local AquaUI = {}

-- Services
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local SoundService = game:GetService("SoundService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Colors (Light Blue Premium Theme)
local Colors = {
    Primary = Color3.fromRGB(173, 216, 230),   -- #ADD8E6
    Secondary = Color3.fromRGB(135, 206, 235), -- #87CEEB
    Accent = Color3.fromRGB(70, 130, 180),     -- #4682B4
    Darker = Color3.fromRGB(30, 58, 138),      -- #1E3A8A
    Background = Color3.fromRGB(10, 10, 25),
    Text = Color3.fromRGB(240, 245, 255),
    Success = Color3.fromRGB(16, 185, 129),
    Warning = Color3.fromRGB(245, 158, 11),
    Danger = Color3.fromRGB(239, 68, 68),
    Black = Color3.fromRGB(0, 0, 0),
    White = Color3.fromRGB(255, 255, 255)
}

-- Tween Presets
local Tweens = {
    Smooth = TweenInfo.new(0.25, Enum.EasingStyle.Quint, Enum.EasingDirection.Out),
    Fast = TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
    Entrance = TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out, 0, false, 0.1),
    Bounce = TweenInfo.new(0.35, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out)
}

-- Utility Functions
local function Create(class, props)
    local obj = Instance.new(class)
    for prop, val in pairs(props) do
        if prop ~= "Parent" then
            if pcall(function() return obj[prop] end) then
                obj[prop] = val
            end
        end
    end
    return obj
end

local function ApplyGlass(frame)
    -- Corner
    local corner = Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = frame
    })
    
    -- Stroke
    local stroke = Create("UIStroke", {
        Thickness = 1.5,
        Color = Colors.Accent,
        Transparency = 0.5,
        Parent = frame
    })
    
    -- Gradient
    local gradient = Create("UIGradient", {
        Rotation = 90,
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Colors.Primary),
            ColorSequenceKeypoint.new(0.5, Colors.Secondary),
            ColorSequenceKeypoint.new(1, Colors.Darker)
        }),
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.7),
            NumberSequenceKeypoint.new(1, 0.9)
        }),
        Parent = frame
    })
    
    return {corner, stroke, gradient}
end

local function PlaySound(id)
    if not AquaUI.SoundsEnabled then return end
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://" .. tostring(id)
        sound.Volume = 0.3
        sound.Parent = SoundService
        sound:Play()
        game.Debris:AddItem(sound, 2)
    end)
end

-- Initialize
AquaUI.SoundsEnabled = true
AquaUI.Windows = {}

-- Main Window Creator
function AquaUI:CreateWindow(title, size, position)
    local Window = {}
    Window.Tabs = {}
    Window.ActiveTab = nil
    
    -- ScreenGui
    local ScreenGui = Create("ScreenGui", {
        Name = "AquaUI_" .. tick(),
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Global
    })
    
    if gethui then
        ScreenGui.Parent = gethui()
    elseif syn and syn.protect_gui then
        syn.protect_gui(ScreenGui)
        ScreenGui.Parent = game.CoreGui
    else
        ScreenGui.Parent = game.CoreGui
    end
    
    -- Main Container
    local MainFrame = Create("Frame", {
        Size = size or UDim2.new(0, 500, 0, 400),
        Position = position or UDim2.new(0.5, -250, 0.5, -200),
        BackgroundColor3 = Colors.Background,
        BackgroundTransparency = 0.85,
        BorderSizePixel = 0,
        Parent = ScreenGui
    })
    
    ApplyGlass(MainFrame)
    
    -- Title Bar
    local TitleBar = Create("Frame", {
        Size = UDim2.new(1, 0, 0, 40),
        BackgroundColor3 = Colors.Darker,
        BackgroundTransparency = 0.85,
        BorderSizePixel = 0,
        Parent = MainFrame
    })
    
    ApplyGlass(TitleBar)
    
    local TitleText = Create("TextLabel", {
        Size = UDim2.new(1, -80, 1, 0),
        Position = UDim2.new(0, 10, 0, 0),
        BackgroundTransparency = 1,
        Text = title or "AquaUI",
        TextColor3 = Colors.Text,
        TextSize = 18,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = TitleBar
    })
    
    -- Close Button
    local CloseBtn = Create("TextButton", {
        Size = UDim2.new(0, 30, 0, 30),
        Position = UDim2.new(1, -35, 0.5, -15),
        BackgroundColor3 = Colors.Danger,
        BackgroundTransparency = 0.8,
        Text = "×",
        TextColor3 = Colors.Text,
        TextSize = 20,
        Font = Enum.Font.GothamBold,
        AutoButtonColor = false,
        Parent = TitleBar
    })
    
    ApplyGlass(CloseBtn)
    
    CloseBtn.MouseButton1Click:Connect(function()
        PlaySound(1873903401)
        ScreenGui:Destroy()
        for i, v in pairs(AquaUI.Windows) do
            if v == Window then
                table.remove(AquaUI.Windows, i)
                break
            end
        end
    end)
    
    -- Tab Container
    local TabContainer = Create("Frame", {
        Size = UDim2.new(0, 120, 1, -40),
        Position = UDim2.new(0, 0, 0, 40),
        BackgroundColor3 = Colors.Darker,
        BackgroundTransparency = 0.9,
        BorderSizePixel = 0,
        Parent = MainFrame
    })
    
    local TabLayout = Create("UIListLayout", {
        Padding = UDim.new(0, 5),
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Parent = TabContainer
    })
    
    local TabPadding = Create("UIPadding", {
        PaddingTop = UDim.new(0, 10),
        Parent = TabContainer
    })
    
    -- Content Container
    local ContentContainer = Create("Frame", {
        Size = UDim2.new(1, -120, 1, -40),
        Position = UDim2.new(0, 120, 0, 40),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Parent = MainFrame
    })
    
    -- Make draggable
    local dragging, dragInput, dragStart, startPos
    
    local function Update(input)
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
    
    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    
    TitleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            Update(input)
        end
    end)
    
    -- Entrance animation
    MainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    MainFrame.Position = UDim2.new(0.5, 0, 0.4, 0)
    MainFrame.BackgroundTransparency = 1
    
    local entrance = TweenService:Create(MainFrame, Tweens.Entrance, {
        BackgroundTransparency = 0.85,
        Position = position or UDim2.new(0.5, -250, 0.5, -200)
    })
    entrance:Play()
    
    PlaySound(3333962289)
    
    -- Window Methods
    function Window:AddTab(name)
        local Tab = {}
        Tab.Name = name
        Tab.Sections = {}
        
        -- Tab Button
        local TabButton = Create("TextButton", {
            Size = UDim2.new(0.9, 0, 0, 35),
            BackgroundColor3 = Colors.Primary,
            BackgroundTransparency = 0.85,
            Text = name,
            TextColor3 = Colors.Text,
            TextSize = 14,
            Font = Enum.Font.Gotham,
            AutoButtonColor = false,
            Parent = TabContainer
        })
        
        ApplyGlass(TabButton)
        
        -- Tab Content
        local TabContent = Create("ScrollingFrame", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = Colors.Accent,
            Visible = false,
            Parent = ContentContainer
        })
        
        local ContentLayout = Create("UIListLayout", {
            Padding = UDim.new(0, 15),
            HorizontalAlignment = Enum.HorizontalAlignment.Center,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Parent = TabContent
        })
        
        local ContentPadding = Create("UIPadding", {
            PaddingTop = UDim.new(0, 20),
            PaddingLeft = UDim.new(0, 20),
            PaddingRight = UDim.new(0, 20),
            Parent = TabContent
        })
        
        ContentLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            TabContent.CanvasSize = UDim2.new(0, 0, 0, ContentLayout.AbsoluteContentSize.Y + 40)
        end)
        
        -- Select first tab
        if #Window.Tabs == 0 then
            Window.ActiveTab = Tab
            TabButton.BackgroundTransparency = 0.7
            TabContent.Visible = true
        end
        
        -- Tab button click
        TabButton.MouseButton1Click:Connect(function()
            PlaySound(2789713373)
            
            -- Deselect all tabs
            for _, t in pairs(Window.Tabs) do
                t.Button.BackgroundTransparency = 0.85
                t.Content.Visible = false
            end
            
            -- Select this tab
            TabButton.BackgroundTransparency = 0.7
            TabContent.Visible = true
            Window.ActiveTab = Tab
        end)
        
        -- Hover effects
        TabButton.MouseEnter:Connect(function()
            if Window.ActiveTab ~= Tab then
                TweenService:Create(TabButton, Tweens.Smooth, {
                    BackgroundTransparency = 0.75,
                    Size = UDim2.new(0.95, 0, 0, 35)
                }):Play()
            end
        end)
        
        TabButton.MouseLeave:Connect(function()
            if Window.ActiveTab ~= Tab then
                TweenService:Create(TabButton, Tweens.Smooth, {
                    BackgroundTransparency = 0.85,
                    Size = UDim2.new(0.9, 0, 0, 35)
                }):Play()
            end
        end)
        
        Tab.Button = TabButton
        Tab.Content = TabContent
        
        -- Tab Methods
        function Tab:AddSection(title)
            local Section = {}
            Section.Elements = {}
            
            local SectionFrame = Create("Frame", {
                Size = UDim2.new(1, -10, 0, 40),
                BackgroundColor3 = Colors.Darker,
                BackgroundTransparency = 0.9,
                BorderSizePixel = 0,
                LayoutOrder = #Tab.Sections + 1,
                Parent = TabContent
            })
            
            ApplyGlass(SectionFrame)
            
            local TitleLabel = Create("TextLabel", {
                Size = UDim2.new(1, -20, 1, 0),
                Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1,
                Text = title,
                TextColor3 = Colors.Text,
                TextSize = 16,
                Font = Enum.Font.GothamBold,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = SectionFrame
            })
            
            local SectionContent = Create("Frame", {
                Size = UDim2.new(1, 0, 0, 0),
                Position = UDim2.new(0, 0, 1, 0),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Parent = SectionFrame
            })
            
            local SectionLayout = Create("UIListLayout", {
                Padding = UDim.new(0, 8),
                SortOrder = Enum.SortOrder.LayoutOrder,
                Parent = SectionContent
            })
            
            SectionLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
                SectionContent.Size = UDim2.new(1, 0, 0, SectionLayout.AbsoluteContentSize.Y)
                SectionFrame.Size = UDim2.new(1, -10, 0, 40 + SectionLayout.AbsoluteContentSize.Y)
            end)
            
            Section.Frame = SectionFrame
            Section.Content = SectionContent
            
            -- Section Methods
            function Section:AddButton(text, callback)
                local Button = Create("TextButton", {
                    Size = UDim2.new(1, 0, 0, 32),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    AutoButtonColor = false,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(Button)
                
                local ogSize = Button.Size
                
                Button.MouseButton1Click:Connect(function()
                    PlaySound(374218581)
                    
                    -- Click animation
                    TweenService:Create(Button, Tweens.Fast, {
                        Size = UDim2.new(0.96, 0, 0.96, 0)
                    }):Play()
                    
                    task.wait(0.12)
                    TweenService:Create(Button, Tweens.Smooth, {
                        Size = ogSize
                    }):Play()
                    
                    if callback then
                        pcall(callback)
                    end
                end)
                
                Button.MouseEnter:Connect(function()
                    TweenService:Create(Button, Tweens.Smooth, {
                        BackgroundTransparency = 0.75,
                        Size = UDim2.new(1.03, 0, 1.03, 0)
                    }):Play()
                end)
                
                Button.MouseLeave:Connect(function()
                    TweenService:Create(Button, Tweens.Smooth, {
                        BackgroundTransparency = 0.85,
                        Size = ogSize
                    }):Play()
                end)
                
                table.insert(Section.Elements, Button)
                return Button
            end
            
            function Section:AddToggle(text, default, callback)
                local Toggle = {Value = default or false}
                
                local ToggleFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 32),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(ToggleFrame)
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(0.7, 0, 1, 0),
                    Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = ToggleFrame
                })
                
                local ToggleButton = Create("Frame", {
                    Size = UDim2.new(0, 40, 0, 20),
                    Position = UDim2.new(1, -50, 0.5, -10),
                    BackgroundColor3 = Toggle.Value and Colors.Success or Colors.Danger,
                    BackgroundTransparency = 0.7,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Parent = ToggleFrame
                })
                
                Create("UICorner", {
                    CornerRadius = UDim.new(1, 0),
                    Parent = ToggleButton
                })
                
                local ToggleCircle = Create("Frame", {
                    Size = UDim2.new(0, 14, 0, 14),
                    Position = Toggle.Value and UDim2.new(1, -17, 0.5, -7) or UDim2.new(0, 3, 0.5, -7),
                    BackgroundColor3 = Colors.White,
                    BackgroundTransparency = 0.8,
                    AnchorPoint = Vector2.new(0.5, 0.5),
                    Parent = ToggleButton
                })
                
                Create("UICorner", {
                    CornerRadius = UDim.new(1, 0),
                    Parent = ToggleCircle
                })
                
                local function UpdateToggle()
                    TweenService:Create(ToggleButton, Tweens.Smooth, {
                        BackgroundColor3 = Toggle.Value and Colors.Success or Colors.Danger
                    }):Play()
                    
                    TweenService:Create(ToggleCircle, Tweens.Smooth, {
                        Position = Toggle.Value and UDim2.new(1, -17, 0.5, -7) or UDim2.new(0, 3, 0.5, -7)
                    }):Play()
                    
                    if callback then
                        pcall(callback, Toggle.Value)
                    end
                end
                
                ToggleFrame.MouseButton1Click:Connect(function()
                    PlaySound(2789713373)
                    Toggle.Value = not Toggle.Value
                    UpdateToggle()
                end)
                
                UpdateToggle()
                
                Toggle.Toggle = function(value)
                    if value ~= nil then
                        Toggle.Value = value
                    else
                        Toggle.Value = not Toggle.Value
                    end
                    UpdateToggle()
                end
                
                table.insert(Section.Elements, ToggleFrame)
                return Toggle
            end
            
            function Section:AddSlider(text, min, max, increment, default, callback)
                local Slider = {
                    Value = default or min,
                    Min = min,
                    Max = max
                }
                
                local SliderFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 50),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(SliderFrame)
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(1, -20, 0, 20),
                    Position = UDim2.new(0, 10, 0, 5),
                    BackgroundTransparency = 1,
                    Text = text .. ": " .. tostring(Slider.Value),
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = SliderFrame
                })
                
                local Track = Create("Frame", {
                    Size = UDim2.new(1, -20, 0, 4),
                    Position = UDim2.new(0, 10, 1, -15),
                    BackgroundColor3 = Colors.Darker,
                    BackgroundTransparency = 0.7,
                    BorderSizePixel = 0,
                    Parent = SliderFrame
                })
                
                Create("UICorner", {
                    CornerRadius = UDim.new(1, 0),
                    Parent = Track
                })
                
                local Fill = Create("Frame", {
                    Size = UDim2.new(0, 0, 1, 0),
                    BackgroundColor3 = Colors.Secondary,
                    BackgroundTransparency = 0.5,
                    BorderSizePixel = 0,
                    Parent = Track
                })
                
                Create("UICorner", {
                    CornerRadius = UDim.new(1, 0),
                    Parent = Fill
                })
                
                local Thumb = Create("Frame", {
                    Size = UDim2.new(0, 16, 0, 16),
                    Position = UDim2.new(0, 0, 0.5, -8),
                    BackgroundColor3 = Colors.Accent,
                    BackgroundTransparency = 0.6,
                    AnchorPoint = Vector2.new(0.5, 0.5),
                    Parent = Track
                })
                
                Create("UICorner", {
                    CornerRadius = UDim.new(1, 0),
                    Parent = Thumb
                })
                
                local function UpdateSlider(value)
                    Slider.Value = math.clamp(value, min, max)
                    local percent = (Slider.Value - min) / (max - min)
                    
                    Fill.Size = UDim2.new(percent, 0, 1, 0)
                    Thumb.Position = UDim2.new(percent, 0, 0.5, -8)
                    Label.Text = text .. ": " .. string.format("%.2f", Slider.Value)
                    
                    if callback then
                        pcall(callback, Slider.Value)
                    end
                end
                
                local dragging = false
                
                local function InputChanged(input)
                    if dragging then
                        local pos = (input.Position.X - Track.AbsolutePosition.X) / Track.AbsoluteSize.X
                        local value = min + (max - min) * math.clamp(pos, 0, 1)
                        
                        if increment then
                            value = math.floor(value / increment) * increment
                        end
                        
                        UpdateSlider(value)
                    end
                end
                
                Thumb.InputBegan:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = true
                        PlaySound(2789713373)
                    end
                end)
                
                Thumb.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = false
                    end
                end)
                
                Track.InputBegan:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = true
                        local pos = (input.Position.X - Track.AbsolutePosition.X) / Track.AbsoluteSize.X
                        local value = min + (max - min) * math.clamp(pos, 0, 1)
                        
                        if increment then
                            value = math.floor(value / increment) * increment
                        end
                        
                        UpdateSlider(value)
                        PlaySound(2789713373)
                    end
                end)
                
                Track.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = false
                    end
                end)
                
                UserInputService.InputChanged:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
                        InputChanged(input)
                    end
                end)
                
                UpdateSlider(default or min)
                
                Slider.Set = function(value)
                    UpdateSlider(value)
                end
                
                table.insert(Section.Elements, SliderFrame)
                return Slider
            end
            
            function Section:AddDropdown(text, options, default, callback)
                local Dropdown = {
                    Value = default or options[1],
                    Options = options,
                    Open = false
                }
                
                local DropdownFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 32),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(DropdownFrame)
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(0.7, 0, 1, 0),
                    Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = DropdownFrame
                })
                
                local ValueLabel = Create("TextLabel", {
                    Size = UDim2.new(0.25, 0, 1, 0),
                    Position = UDim2.new(0.75, 0, 0, 0),
                    BackgroundTransparency = 1,
                    Text = tostring(Dropdown.Value),
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    Parent = DropdownFrame
                })
                
                local DropdownList = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 0),
                    Position = UDim2.new(0, 0, 1, 5),
                    BackgroundColor3 = Colors.Darker,
                    BackgroundTransparency = 0.9,
                    BorderSizePixel = 0,
                    ClipsDescendants = true,
                    Visible = false,
                    Parent = DropdownFrame
                })
                
                ApplyGlass(DropdownList)
                
                local ListLayout = Create("UIListLayout", {
                    SortOrder = Enum.SortOrder.LayoutOrder,
                    Parent = DropdownList
                })
                
                local function UpdateDropdown()
                    ValueLabel.Text = tostring(Dropdown.Value)
                    if callback then
                        pcall(callback, Dropdown.Value)
                    end
                end
                
                local function ToggleList()
                    Dropdown.Open = not Dropdown.Open
                    DropdownList.Visible = Dropdown.Open
                    
                    if Dropdown.Open then
                        TweenService:Create(DropdownList, Tweens.Smooth, {
                            Size = UDim2.new(1, 0, 0, math.min(#options * 30, 150))
                        }):Play()
                        PlaySound(3333962289)
                    else
                        TweenService:Create(DropdownList, Tweens.Smooth, {
                            Size = UDim2.new(1, 0, 0, 0)
                        }):Play()
                    end
                end
                
                DropdownFrame.MouseButton1Click:Connect(ToggleList)
                
                -- Create options
                for i, option in ipairs(options) do
                    local OptionButton = Create("TextButton", {
                        Size = UDim2.new(1, 0, 0, 30),
                        BackgroundColor3 = Colors.Primary,
                        BackgroundTransparency = 0.9,
                        Text = tostring(option),
                        TextColor3 = Colors.Text,
                        TextSize = 14,
                        Font = Enum.Font.Gotham,
                        AutoButtonColor = false,
                        LayoutOrder = i,
                        Parent = DropdownList
                    })
                    
                    OptionButton.MouseButton1Click:Connect(function()
                        Dropdown.Value = option
                        UpdateDropdown()
                        ToggleList()
                        PlaySound(2789713373)
                    end)
                    
                    OptionButton.MouseEnter:Connect(function()
                        TweenService:Create(OptionButton, Tweens.Smooth, {
                            BackgroundTransparency = 0.8
                        }):Play()
                    end)
                    
                    OptionButton.MouseLeave:Connect(function()
                        TweenService:Create(OptionButton, Tweens.Smooth, {
                            BackgroundTransparency = 0.9
                        }):Play()
                    end)
                end
                
                UpdateDropdown()
                
                Dropdown.Set = function(value)
                    if table.find(options, value) then
                        Dropdown.Value = value
                        UpdateDropdown()
                    end
                end
                
                table.insert(Section.Elements, DropdownFrame)
                return Dropdown
            end
            
            function Section:AddTextbox(text, placeholder, callback)
                local TextboxFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 32),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(TextboxFrame)
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(0.3, 0, 1, 0),
                    Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = TextboxFrame
                })
                
                local TextBox = Create("TextBox", {
                    Size = UDim2.new(0.65, -15, 1, -10),
                    Position = UDim2.new(0.35, 5, 0, 5),
                    BackgroundColor3 = Colors.Darker,
                    BackgroundTransparency = 0.9,
                    Text = "",
                    PlaceholderText = placeholder or "Type here...",
                    TextColor3 = Colors.Text,
                    PlaceholderColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = TextboxFrame
                })
                
                ApplyGlass(TextBox)
                
                TextBox.FocusLost:Connect(function(enterPressed)
                    if enterPressed and callback then
                        pcall(callback, TextBox.Text)
                        PlaySound(2789713373)
                    end
                end)
                
                table.insert(Section.Elements, TextboxFrame)
                return TextBox
            end
            
            function Section:AddLabel(text)
                local LabelFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 24),
                    BackgroundTransparency = 1,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(1, -20, 1, 0),
                    Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = LabelFrame
                })
                
                table.insert(Section.Elements, LabelFrame)
                return Label
            end
            
            function Section:AddKeybind(text, defaultKey, callback)
                local Keybind = {
                    Value = defaultKey or Enum.KeyCode.E,
                    Listening = false
                }
                
                local KeybindFrame = Create("Frame", {
                    Size = UDim2.new(1, 0, 0, 32),
                    BackgroundColor3 = Colors.Primary,
                    BackgroundTransparency = 0.85,
                    LayoutOrder = #Section.Elements + 1,
                    Parent = SectionContent
                })
                
                ApplyGlass(KeybindFrame)
                
                local Label = Create("TextLabel", {
                    Size = UDim2.new(0.7, 0, 1, 0),
                    Position = UDim2.new(0, 10, 0, 0),
                    BackgroundTransparency = 1,
                    Text = text,
                    TextColor3 = Colors.Text,
                    TextSize = 14,
                    Font = Enum.Font.Gotham,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    Parent = KeybindFrame
                })
                
                local KeyLabel = Create("TextButton", {
                    Size = UDim2.new(0.25, 0, 0.7, 0),
                    Position = UDim2.new(0.75, 0, 0.15, 0),
                    BackgroundColor3 = Colors.Darker,
                    BackgroundTransparency = 0.8,
                    Text = tostring(Keybind.Value.Name),
                    TextColor3 = Colors.Text,
                    TextSize = 12,
                    Font = Enum.Font.Gotham,
                    AutoButtonColor = false,
                    Parent = KeybindFrame
                })
                
                ApplyGlass(KeyLabel)
                
                local function UpdateKeybind()
                    KeyLabel.Text = Keybind.Value.Name
                end
                
                local function StartListening()
                    Keybind.Listening = true
                    KeyLabel.Text = "..."
                    KeyLabel.BackgroundColor3 = Colors.Accent
                    
                    local connection
                    connection = UserInputService.InputBegan:Connect(function(input)
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                            Keybind.Value = input.KeyCode
                            Keybind.Listening = false
                            UpdateKeybind()
                            KeyLabel.BackgroundColor3 = Colors.Darker
                            
                            if callback then
                                pcall(callback, Keybind.Value)
                            end
                            
                            connection:Disconnect()
                            PlaySound(2789713373)
                        end
                    end)
                end
                
                KeyLabel.MouseButton1Click:Connect(StartListening)
                UpdateKeybind()
                
                Keybind.Set = function(key)
                    Keybind.Value = key
                    UpdateKeybind()
                end
                
                table.insert(Section.Elements, KeybindFrame)
                return Keybind
            end
            
            table.insert(Tab.Sections, Section)
            return Section
        end
        
        table.insert(Window.Tabs, Tab)
        return Tab
    end
    
    function Window:Destroy()
        ScreenGui:Destroy()
    end
    
    table.insert(AquaUI.Windows, Window)
    return Window
end

-- Notification System
function AquaUI:Notify(title, text, duration, notifType)
    duration = duration or 5
    notifType = notifType or "info"
    
    local colorMap = {
        info = Colors.Primary,
        success = Colors.Success,
        warning = Colors.Warning,
        error = Colors.Danger
    }
    
    local notifColor = colorMap[notifType] or Colors.Primary
    
    -- Create notification container
    if not self.NotifContainer then
        local container = Create("ScreenGui", {
            Name = "AquaUINotifs",
            ResetOnSpawn = false,
            ZIndexBehavior = Enum.ZIndexBehavior.Global
        })
        
        if gethui then
            container.Parent = gethui()
        elseif syn and syn.protect_gui then
            syn.protect_gui(container)
            container.Parent = game.CoreGui
        else
            container.Parent = game.CoreGui
        end
        
        local frame = Create("Frame", {
            Size = UDim2.new(0, 300, 1, -20),
            Position = UDim2.new(1, -320, 0, 20),
            BackgroundTransparency = 1,
            Parent = container
        })
        
        local layout = Create("UIListLayout", {
            Padding = UDim.new(0, 10),
            HorizontalAlignment = Enum.HorizontalAlignment.Right,
            VerticalAlignment = Enum.VerticalAlignment.Bottom,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Parent = frame
        })
        
        self.NotifContainer = frame
    end
    
    local NotifFrame = Create("Frame", {
        Size = UDim2.new(0, 280, 0, 0),
        BackgroundColor3 = Colors.Background,
        BackgroundTransparency = 0.9,
        BorderSizePixel = 0,
        LayoutOrder = 999,
        Parent = self.NotifContainer
    })
    
    ApplyGlass(NotifFrame)
    
    local TitleLabel = Create("TextLabel", {
        Size = UDim2.new(1, -20, 0, 25),
        Position = UDim2.new(0, 10, 0, 10),
        BackgroundTransparency = 1,
        Text = title,
        TextColor3 = notifColor,
        TextSize = 16,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        Parent = NotifFrame
    })
    
    local TextLabel = Create("TextLabel", {
        Size = UDim2.new(1, -20, 0, 0),
        Position = UDim2.new(0, 10, 0, 35),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = Colors.Text,
        TextSize = 14,
        Font = Enum.Font.Gotham,
        TextWrapped = true,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Top,
        Parent = NotifFrame
    })
    
    -- Auto-size
    local textSize = game.TextService:GetTextSize(text, 14, Enum.Font.Gotham, Vector2.new(260, 1000))
    local textHeight = math.min(textSize.Y, 100)
    TextLabel.Size = UDim2.new(1, -20, 0, textHeight)
    NotifFrame.Size = UDim2.new(0, 280, 0, 50 + textHeight)
    
    -- Progress bar
    local ProgressBar = Create("Frame", {
        Size = UDim2.new(1, 0, 0, 3),
        Position = UDim2.new(0, 0, 1, -3),
        BackgroundColor3 = notifColor,
        BackgroundTransparency = 0.5,
        BorderSizePixel = 0,
        Parent = NotifFrame
    })
    
    -- Entrance
    NotifFrame.Position = UDim2.new(1.2, 0, 1, 0)
    NotifFrame.AnchorPoint = Vector2.new(1, 1)
    
    TweenService:Create(NotifFrame, Tweens.Entrance, {
        Position = UDim2.new(1, 0, 1, 0)
    }):Play()
    
    -- Progress animation
    TweenService:Create(ProgressBar, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
        Size = UDim2.new(0, 0, 0, 3)
    }):Play()
    
    PlaySound(3333962289)
    
    -- Auto-remove
    task.delay(duration, function()
        TweenService:Create(NotifFrame, Tweens.Smooth, {
            Position = UDim2.new(1.2, 0, 1, 0),
            BackgroundTransparency = 1
        }):Play()
        
        task.wait(0.3)
        NotifFrame:Destroy()
    end)
    
    return NotifFrame
end

-- HUD Functions
function AquaUI:CreateHUD()
    local HUD = {}
    
    function HUD:CreateHealthBar(parent, maxHealth)
        local HealthBar = {}
        HealthBar.MaxHealth = maxHealth or 100
        HealthBar.CurrentHealth = maxHealth or 100
        
        local Container = Create("Frame", {
            Size = UDim2.new(0, 200, 0, 30),
            BackgroundColor3 = Colors.Darker,
            BackgroundTransparency = 0.85,
            Parent = parent
        })
        
        ApplyGlass(Container)
        
        local Fill = Create("Frame", {
            Size = UDim2.new(1, -10, 0.6, 0),
            Position = UDim2.new(0, 5, 0.2, 0),
            BackgroundColor3 = Colors.Success,
            BackgroundTransparency = 0.5,
            BorderSizePixel = 0,
            Parent = Container
        })
        
        Create("UICorner", {
            CornerRadius = UDim.new(1, 0),
            Parent = Fill
        })
        
        local HealthText = Create("TextLabel", {
            Size = UDim2.new(1, 0, 1, 0),
            BackgroundTransparency = 1,
            Text = tostring(HealthBar.CurrentHealth) .. " / " .. tostring(HealthBar.MaxHealth),
            TextColor3 = Colors.Text,
            TextSize = 14,
            Font = Enum.Font.GothamBold,
            Parent = Container
        })
        
        HealthBar.Frame = Container
        HealthBar.Fill = Fill
        HealthBar.Text = HealthText
        
        function HealthBar:Set(health)
            HealthBar.CurrentHealth = math.clamp(health, 0, HealthBar.MaxHealth)
            local percent = HealthBar.CurrentHealth / HealthBar.MaxHealth
            
            TweenService:Create(HealthBar.Fill, Tweens.Smooth, {
                Size = UDim2.new(percent, -10, 0.6, 0)
            }):Play()
            
            local color = Color3.new(
                2 * (1 - percent),
                2 * percent,
                0
            ):Lerp(Colors.Success, percent)
            
            TweenService:Create(HealthBar.Fill, Tweens.Smooth, {
                BackgroundColor3 = color
            }):Play()
            
            HealthBar.Text.Text = tostring(math.floor(HealthBar.CurrentHealth)) .. " / " .. tostring(HealthBar.MaxHealth)
        end
        
        return HealthBar
    end
    
    function HUD:CreateAmmoCounter(parent, currentAmmo, maxClip, reserve)
        local AmmoCounter = {}
        AmmoCounter.CurrentAmmo = currentAmmo or 30
        AmmoCounter.MaxClip = maxClip or 30
        AmmoCounter.Reserve = reserve or 90
        
        local Container = Create("Frame", {
            Size = UDim2.new(0, 150, 0, 50),
            BackgroundColor3 = Colors.Darker,
            BackgroundTransparency = 0.85,
            Parent = parent
        })
        
        ApplyGlass(Container)
        
        local CurrentText = Create("TextLabel", {
            Size = UDim2.new(0.6, 0, 0.6, 0),
            Position = UDim2.new(0.1, 0, 0.1, 0),
            BackgroundTransparency = 1,
            Text = tostring(AmmoCounter.CurrentAmmo),
            TextColor3 = Colors.Text,
            TextSize = 24,
            Font = Enum.Font.GothamBold,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = Container
        })
        
        local MaxText = Create("TextLabel", {
            Size = UDim2.new(0.3, 0, 0.4, 0),
            Position = UDim2.new(0.7, 0, 0.15, 0),
            BackgroundTransparency = 1,
            Text = "/ " .. tostring(AmmoCounter.MaxClip),
            TextColor3 = Colors.Text,
            TextSize = 16,
            Font = Enum.Font.Gotham,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = Container
        })
        
        local ReserveText = Create("TextLabel", {
            Size = UDim2.new(1, -10, 0.3, 0),
            Position = UDim2.new(0, 5, 0.65, 0),
            BackgroundTransparency = 1,
            Text = "Reserve: " .. tostring(AmmoCounter.Reserve),
            TextColor3 = Colors.Text,
            TextSize = 12,
            Font = Enum.Font.Gotham,
            Parent = Container
        })
        
        AmmoCounter.Frame = Container
        AmmoCounter.CurrentText = CurrentText
        AmmoCounter.MaxText = MaxText
        AmmoCounter.ReserveText = ReserveText
        
        function AmmoCounter:Update(ammo, reserve)
            AmmoCounter.CurrentAmmo = ammo or AmmoCounter.CurrentAmmo
            AmmoCounter.Reserve = reserve or AmmoCounter.Reserve
            
            if AmmoCounter.CurrentAmmo / AmmoCounter.MaxClip < 0.3 then
                TweenService:Create(AmmoCounter.CurrentText, Tweens.Bounce, {
                    TextColor3 = Colors.Warning
                }):Play()
            else
                AmmoCounter.CurrentText.TextColor3 = Colors.Text
            end
            
            AmmoCounter.CurrentText.Text = tostring(AmmoCounter.CurrentAmmo)
            AmmoCounter.ReserveText.Text = "Reserve: " .. tostring(AmmoCounter.Reserve)
            
            TweenService:Create(AmmoCounter.CurrentText, Tweens.Fast, {
                TextSize = 28
            }):Play()
            
            task.wait(0.1)
            
            TweenService:Create(AmmoCounter.CurrentText, Tweens.Smooth, {
                TextSize = 24
            }):Play()
        end
        
        return AmmoCounter
    end
    
    function HUD:CreateKillFeed(parent)
        local KillFeed = {}
        KillFeed.Entries = {}
        
        local Container = Create("Frame", {
            Size = UDim2.new(0, 300, 0.5, 0),
            Position = UDim2.new(0, 20, 0, 20),
            BackgroundColor3 = Colors.Background,
            BackgroundTransparency = 0.9,
            Parent = parent
        })
        
        ApplyGlass(Container)
        
        local ListLayout = Create("UIListLayout", {
            Padding = UDim.new(0, 5),
            SortOrder = Enum.SortOrder.LayoutOrder,
            Parent = Container
        })
        
        KillFeed.Frame = Container
        
        function KillFeed:Add(killer, victim, weapon)
            local EntryFrame = Create("Frame", {
                Size = UDim2.new(1, -10, 0, 30),
                BackgroundColor3 = Colors.Darker,
                BackgroundTransparency = 0.85,
                LayoutOrder = 1,
                Parent = Container
            })
            
            ApplyGlass(EntryFrame)
            
            local KillerText = Create("TextLabel", {
                Size = UDim2.new(0.4, 0, 1, 0),
                Position = UDim2.new(0, 10, 0, 0),
                BackgroundTransparency = 1,
                Text = killer,
                TextColor3 = Colors.Primary,
                TextSize = 14,
                Font = Enum.Font.GothamBold,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = EntryFrame
            })
            
            local Icon = Create("TextLabel", {
                Size = UDim2.new(0, 20, 1, 0),
                Position = UDim2.new(0.4, 0, 0, 0),
                BackgroundTransparency = 1,
                Text = "→",
                TextColor3 = Colors.Danger,
                TextSize = 16,
                Font = Enum.Font.GothamBold,
                Parent = EntryFrame
            })
            
            local VictimText = Create("TextLabel", {
                Size = UDim2.new(0.4, 0, 1, 0),
                Position = UDim2.new(0.6, -20, 0, 0),
                BackgroundTransparency = 1,
                Text = victim,
                TextColor3 = Colors.Text,
                TextSize = 14,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                Parent = EntryFrame
            })
            
            local WeaponText = Create("TextLabel", {
                Size = UDim2.new(0.2, 0, 1, 0),
                Position = UDim2.new(0.8, 0, 0, 0),
                BackgroundTransparency = 1,
                Text = weapon or "Gun",
                TextColor3 = Colors.Accent,
                TextSize = 12,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Right,
                Parent = EntryFrame
            })
            
            EntryFrame.Position = UDim2.new(-1, 0, 0, 0)
            TweenService:Create(EntryFrame, Tweens.Smooth, {
                Position = UDim2.new(0, 0, 0, 0)
            }):Play()
            
            table.insert(KillFeed.Entries, 1, EntryFrame)
            
            if #KillFeed.Entries > 6 then
                local old = KillFeed.Entries[#KillFeed.Entries]
                TweenService:Create(old, Tweens.Smooth, {
                    Position = UDim2.new(1, 0, 0, 0),
                    BackgroundTransparency = 1
                }):Play()
                
                task.wait(0.3)
                old:Destroy()
                table.remove(KillFeed.Entries, #KillFeed.Entries)
            end
            
            task.delay(8, function()
                TweenService:Create(EntryFrame, Tweens.Smooth, {
                    BackgroundTransparency = 1
                }):Play()
                
                task.wait(0.3)
                EntryFrame:Destroy()
                
                for i, v in pairs(KillFeed.Entries) do
                    if v == EntryFrame then
                        table.remove(KillFeed.Entries, i)
                        break
                    end
                end
            end)
            
            return EntryFrame
        end
        
        return KillFeed
    end
    
    return HUD
end

-- Utility
function AquaUI:ToggleSounds(enabled)
    self.SoundsEnabled = enabled
    return self.SoundsEnabled
end

-- Example usage for executors:
--[[
local AquaUI = loadstring(game:HttpGet("https://pastebin.com/raw/..."))()

local window = AquaUI:CreateWindow("AquaUI Premium", UDim2.new(0, 500, 0, 400))

local aimTab = window:AddTab("Aimbot")
local visTab = window:AddTab("Visuals")
local miscTab = window:AddTab("Misc")

local aimSection = aimTab:AddSection("Aimbot Settings")
local visSection = visTab:AddSection("ESP")
local miscSection = miscTab:AddSection("Utilities")

aimSection:AddToggle("Enable Aimbot", false, function(state)
    print("Aimbot:", state)
end)

aimSection:AddSlider("FOV", 1, 360, 1, 90, function(value)
    print("FOV:", value)
end)

visSection:AddDropdown("Box Type", {"2D", "3D", "Corner"}, "2D", function(option)
    print("Box type:", option)
end)

miscSection:AddButton("Print Hello", function()
    print("Hello from AquaUI!")
end)

AquaUI:Notify("Success", "AquaUI loaded!", 3, "success")
]]
local window = AquaUI:CreateWindow("AquaUI Premium", UDim2.new(0, 500, 0, 400))

local aimTab = window:AddTab("Aimbot")
local visTab = window:AddTab("Visuals")
local miscTab = window:AddTab("Misc")

local aimSection = aimTab:AddSection("Aimbot Settings")
local visSection = visTab:AddSection("ESP")
local miscSection = miscTab:AddSection("Utilities")

aimSection:AddToggle("Enable Aimbot", false, function(state)
    print("Aimbot:", state)
end)

aimSection:AddSlider("FOV", 1, 360, 1, 90, function(value)
    print("FOV:", value)
end)

visSection:AddDropdown("Box Type", {"2D", "3D", "Corner"}, "2D", function(option)
    print("Box type:", option)
end)

miscSection:AddButton("Print Hello", function()
    print("Hello from AquaUI!")
end)

AquaUI:Notify("Success", "AquaUI loaded!", 3, "success")
]]


return AquaUI


