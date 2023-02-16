# swiftui_dismissable_multiple_image_viewer_studying_gestures...


```swift

//
//  ContentView.swift
//  SwipeDismissTutorial
//
//  Created by paige shin on 2023/02/17.
//

import SwiftUI

class HomeViewModel: ObservableObject {
    
    @Published var data: [String] = [
        "1", "2", "3", "4", "5"
    ]
    
    // Properteis For Image Viewer
    @Published var showImageViewer: Bool = false
    
    @Published var selectedImageID: String = ""
    

}

struct ContentView: View {
        
    @StateObject var vm = HomeViewModel()
    var body: some View {
        ScrollView {
            HStack(alignment: .top, spacing: 15) {
                Image("logo")
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(width: 60, height: 60)
                    .clipShape(Circle())
                
                VStack(alignment: .leading, spacing: 10, content: {
                    
                    // In SwiftUI We can concatenate two ore more Text's...
                    (
                    
                        Text("PaigeSoftware")
                            .fontWeight(.bold)
                        
                        +
                        
                        Text("@_Paigesoftware")
                            .foregroundColor(.gray)
                        
                    )
                    
                    Text("#ios #swiftui #paigesoftware")
                        .foregroundColor(.blue)
                    
                    Text("SungHee New Photos :)))))")
                    
                    // Our Custom Grid of Items.....
                    
                    // Since we having only two columns in a row...
                    // and max is 4 Grid Boxes....
                    let columns = Array(repeating: GridItem(.flexible(), spacing: 15), count: 2)
                    
                    LazyVGrid(columns: columns, alignment: .leading) {
                        ForEach(self.vm.data.indices, id: \.self) { index in
                            GridImageView(index: index)
                                
                        }
                    } //: LAZYVGRID
                    
                }) //: VSTACK
                
            } //: HSTACK
            .padding()
            .frame(maxWidth: .infinity, alignment: .leading)
        } //: SCROLLVIEW
        .overlay(
            // Image Viewer...
            ZStack {
                if self.vm.showImageViewer {
                    
                    ImageView()
                }
            } //: ZSACK
        )
        .environmentObject(self.vm)
        
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct GridImageView: View {
    
    @EnvironmentObject var vm: HomeViewModel
    var index: Int
    
    var body: some View {
        Button {
            
            withAnimation(.easeInOut) {
                self.vm.showImageViewer.toggle()
                // For Page Tab View Automatic Scrolling...
                self.vm.selectedImageID = self.vm.data[self.index]
            }
            
        } label: {
            ZStack {
                
                if self.index <= 3 {
                    Image(self.vm.data[self.index])
                        .resizable()
                        .aspectRatio(contentMode: .fill)
                        .frame(width: self.getWidth(index: self.index), height: 120, alignment: .center)
                        .cornerRadius(12)
                }
                
                // Showing the count of remaining images
                if self.vm.data.count > 4 && self.index == 3{
                    RoundedRectangle(cornerRadius: 12)
                        .fill(.black.opacity(0.3))
                    
                    let remainingImages: Int = self.vm.data.count - 4
                    Text("+\(remainingImages)")
                        .font(.title)
                        .fontWeight(.heavy)
                        .foregroundColor(.white)
                }
                
            } //: ZSTACK
        } //: BUTTON
    }
    
    // expanding Image Size when space is available...
    
    func getWidth(index: Int) -> CGFloat {
        
        let width: CGFloat = self.getRect().width - 100
        
        if self.vm.data.count % 2 == 0 {
            return width / 2
        } else {
            if index == self.vm.data.count - 1 {
                return width
            } else {
                return width / 2
            }
        }
        
    }
    
}

// extending view to get screen size

extension View {
    
    func getRect() -> CGRect {
        UIScreen.main.bounds
    }
    
}


struct ImageView: View {
    
    @EnvironmentObject var vm: HomeViewModel
    
    // Since onChange has a problem in Drage Gesture....
    @GestureState var draggingOffset: CGSize = .zero
    @State var bgOpacity: Double = 1
    @State var imageScale: CGFloat = 1
    
    var body: some View {
        ZStack {

            Color
                .black
                .opacity(self.bgOpacity)
                .ignoresSafeArea()
            
            TabView(selection: self.$vm.selectedImageID) {
                ForEach(self.vm.data, id: \.self) { image in
                    Image(image)
                        .resizable()
                        .aspectRatio(contentMode: .fit)
                        .tag(image)
                        .scaleEffect(self.vm.selectedImageID == image ? (self.imageScale > 1 ? self.imageScale : 1) : 1)
                        .gesture(
                            // Magnifying Gesture...
                            MagnificationGesture()
                                .onChanged({ value in
                                    self.imageScale = value
                                })
                                .onEnded({ _ in
                                    withAnimation(.spring()) {
                                        self.imageScale = 1
                                    }
                                })
                            // Double tab to zoom..
                                .simultaneously(with: TapGesture(count: 2))
                                .onEnded({ value in
                                    withAnimation {
                                        self.imageScale = self.imageScale > 1 ? 1 : 4
                                    }
                                })
                        )
                    
                    // MARK: DRAG, Moving Only Image...
                    // For Smooth Animation ...
//                        .offset(self.vm.imageViewerOffset)
                        .offset(y: self.draggingOffset.height)
                    
                } //: FOREACH
            } //: TABVIEW
            .tabViewStyle(PageTabViewStyle(indexDisplayMode: .always))
            .overlay(
                Button(action: {
                    withAnimation(.default) {
                        self.vm.showImageViewer.toggle()
                    }
                }, label: {
                    Image(systemName: "xmark")
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.white.opacity(0.35))
                        .clipShape(Circle())
                })
                .opacity(self.bgOpacity)
                .padding(10)
                ,alignment: .topTrailing
            )
        } //: ZSTACK
        // MARK: DRAG
        .gesture(
            DragGesture()
                .updating(self.$draggingOffset, body: { value, outValue, _ in
                    outValue = value.translation
                    self.onChange(value: self.draggingOffset)
                })
                .onEnded({ value in
                    self.onEnd(value: value)
                })
        )
        .transition(.move(edge: .bottom))
    } //: BODY

    func onChange(value: CGSize) {
        // calculating opactiy...
        let halfHeight: CGFloat = UIScreen.main.bounds.height / 2
        let progress: CGFloat = self.draggingOffset.height / halfHeight
            
        withAnimation(.default) {
            DispatchQueue.main.async {
//                self.bgOpacity = Double(1 - progress)
                self.bgOpacity = Double(1 - (progress < 0 ? -progress : progress))
            }
        }
        
    }
    
    func onEnd(value: DragGesture.Value) {
        withAnimation(.easeInOut) {
            let translation: CGFloat = value.translation.height
            
//            if translation < 0 {
//                translation = -translation
//            }
            
            if translation < 250 {
                self.bgOpacity = 1
            } else {
                self.vm.showImageViewer = false
                self.bgOpacity = 1
            }
        }
    }
    
}


```
