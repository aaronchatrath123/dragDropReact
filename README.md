```tsx
const simulateWidgetDrop = () => {
    const dataTransfer = new DataTransfer()
    Object.defineProperty(dataTransfer, 'getData', {
      value: (type: string) => type === 'widget' ? 'AreaChart' : ''
    })

    cy.get('[data-cy="display-container"]')
      .trigger('drop', { dataTransfer })
      .wait(500) // Wait for drop to process
  }
```

```tsx
// Helper function to handle drag and drop
  const performDragAndDrop = () => {
    const mockDataTransfer = {
      getData: (key: string) => key === 'widget' ? 'AreaChart' : '',
      setData: (type: string, value: string) => {},
      effectAllowed: 'move',
      dropEffect: 'move',
      types: ['widget'],
      items: [],
      files: [],
      clearData: () => {}
    }

    // Show drag source and start drag
    cy.get('[data-cy="drag-source"]')
      .invoke('show')
      .trigger('mousedown')
      .trigger('dragstart', {
        dataTransfer: {
          ...mockDataTransfer,
          setData: (type: string, value: string) => {
            expect(type).to.equal('widget')
            expect(value).to.equal('AreaChart')
          }
        }
      })

    // Move to drop target and drop
    cy.get('[data-cy="display-container"]')
      .trigger('dragenter', { dataTransfer: mockDataTransfer })
      .trigger('dragover', { 
        dataTransfer: mockDataTransfer,
        clientX: 100,
        clientY: 100
      })
      .trigger('drop', { dataTransfer: mockDataTransfer })

    // End drag and hide source
    cy.get('[data-cy="drag-source"]')
      .trigger('dragend', { dataTransfer: mockDataTransfer })
      .invoke('hide')
      .wait(500) // Wait for drop to process
  }

```

```tsx

 <div 
            data-cy="drag-source" 
            style={{ 
              width: '100px', 
              height: '100px', 
              position: 'absolute',
              top: 0,
              left: 0 
            }}
          />
  const mockDataTransfer = {
      getData: (key: string) => key === 'widget' ? 'AreaChart' : '',
      setData: (type: string, value: string) => {},
      effectAllowed: 'move',
      dropEffect: 'move',
      types: ['widget'],
      items: [],
      files: [],
      clearData: () => {}
    }

    // Start drag from source
    cy.get('[data-cy="drag-source"]')
      .trigger('mousedown')
      .trigger('dragstart', {
        dataTransfer: {
          ...mockDataTransfer,
          setData: (type: string, value: string) => {
            expect(type).to.equal('widget')
            expect(value).to.equal('AreaChart')
          }
        }
      })

    // Move to drop target and drop
    cy.get('[data-cy="display-container"]')
      .trigger('dragenter', { dataTransfer: mockDataTransfer })
      .trigger('dragover', { 
        dataTransfer: mockDataTransfer,
        clientX: 100,
        clientY: 100
      })
      .trigger('drop', { dataTransfer: mockDataTransfer })

    // End drag on source
    cy.get('[data-cy="drag-source"]')
      .trigger('dragend', { dataTransfer: mockDataTransfer })
      .wait(500) // Wait for widget to be added

    // Find the widget and trigger hover
    cy.get('.react-grid-item')
      .first()
      .trigger('mouseover')
      .trigger('mouseenter')
      .wait(300) // Wait for fade animation

    // Now the controls should be visible
    cy.get('.widget-controls')
      .should('be.visible')
      .within(() => {
        cy.get('[data-cy^="delete-widget-"]')
          .should('be.visible')
          .click()
      })

    // Confirm deletion in dialog
    cy.get('[data-cy="delete-dialog"]')
      .should('be.visible')
    
    cy.contains('button', 'Delete')
      .should('be.visible')
      .click()

    // Verify empty state is shown again
    cy.get('[data-cy="empty-state"]')
      .should('be.visible')
  })
   

```

```tsx
   <Box className={`settings-container ${isExpanded ? 'expanded' : 'collapsed'}`}>
      <Box className={`settings-expanded-panel ${isExpanded ? 'expanded' : 'collapsed'}`}>
        <Box className={`settings-header ${isExpanded ? 'expanded' : 'collapsed'}`}>
          <IconButton
            onClick={() => setIsExpanded(false)}
            size="small"
            className="collapse-button"
          >
            <ArrowForwardIosIcon sx={{ fontSize: 18 }} />
          </IconButton>
          <Typography variant="h6">
            Available Widgets
          </Typography>
        </Box>

        <List className="widget-list">
          {widgets.map((widget, index) => {
            const isSelected = selectedWidgets.has(widget.type);
            return (
              <ListItem
                key={widget.type}
                draggable={!isSelected}
                onDragStart={(e) => handleDragStart(e, widget.type)}
                component={Paper}
                className={`widget-list-item ${isSelected ? 'selected' : ''} ${!isExpanded ? 'collapsed' : ''}`}
                style={{ transitionDelay: `${index * 0.05}s` }}
              >
                <ListItemIcon className={`widget-icon ${isSelected ? 'selected' : ''}`}>
                  {widget.icon}
                </ListItemIcon>
                <ListItemText
                  primary={widget.label}
                  secondary={isSelected ? 'Already added to dashboard' : widget.description}
                  primaryTypographyProps={{
                    fontWeight: 'medium',
                    color: isSelected ? 'text.disabled' : 'text.primary'
                  }}
                />
              </ListItem>
            );
          })}
        </List>
      </Box>

      <Box className={`settings-collapsed-panel ${isExpanded ? 'expanded' : 'collapsed'}`}>
        <IconButton
          onClick={() => setIsExpanded(true)}
          size="small"
          className={`expand-button ${isExpanded ? 'expanded' : 'collapsed'}`}
        >
          <ArrowBackIosNewIcon sx={{ fontSize: 20 }} />
        </IconButton>

        <Box className="collapsed-widgets-container">
          {widgets.map((widget, index) => {
            const isSelected = selectedWidgets.has(widget.type);
            return (
              <Box
                key={widget.type}
                className={`${isExpanded ? 'expanded' : 'collapsed'}`}
                style={{ transitionDelay: `${index * 0.05}s` }}
              >
                {renderTooltip(widget, isSelected)}
              </Box>
            );
          })}
        </Box>
      </Box>
    </Box>

```


```tsx
.settings-container {
  height: 100%;
  display: flex;
  flex-direction: row;
  position: relative;
  transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.expanded {
    width: 250px;
  }

  &.collapsed {
    width: 64px;
  }
}

.settings-expanded-panel {
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  width: 250px;
  display: flex;
  flex-direction: column;
  padding: 16px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.expanded {
    opacity: 1;
    visibility: visible;
    transform: translateX(0);
  }

  &.collapsed {
    opacity: 0;
    visibility: hidden;
    transform: translateX(50px);
  }
}

.settings-header {
  display: flex;
  align-items: center;
  margin-bottom: 16px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.expanded {
    opacity: 1;
    transform: translateX(0);
  }

  &.collapsed {
    opacity: 0;
    transform: translateX(-20px);
  }
}

.collapse-button {
  margin-right: 8px;
  color: var(--text-secondary);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

  &:hover {
    color: var(--primary-main);
    transform: scale(1.1);
  }
}

.widget-list {
  flex: 1;
  overflow-y: auto;

  &::-webkit-scrollbar {
    width: 8px;
  }

  &::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 4px;
  }

  &::-webkit-scrollbar-thumb {
    background: #888;
    border-radius: 4px;

    &:hover {
      background: #666;
    }
  }
}

.widget-list-item {
  margin-bottom: 16px;
  border-radius: 4px;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.selected {
    cursor: not-allowed;
    opacity: 0.5;
    background-color: var(--action-disabled-background);
  }

  &:not(.selected) {
    cursor: grab;
    background-color: var(--background-paper);

    &:hover {
      background-color: var(--action-hover);
      transform: translateY(-2px);
    }
  }

  &.collapsed {
    opacity: 0;
    transform: translateX(-20px);
  }
}

.widget-icon {
  &.selected {
    color: var(--action-disabled);
  }

  &:not(.selected) {
    color: var(--primary-main);
  }
}

.settings-collapsed-panel {
  position: absolute;
  right: 0;
  top: 0;
  bottom: 0;
  width: 64px;
  display: flex;
  flex-direction: column;
  align-items: center;
  background-color: var(--background-paper);
  padding: 16px 0;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.expanded {
    opacity: 0;
    visibility: hidden;
    transform: translateX(-50px);
  }

  &.collapsed {
    opacity: 1;
    visibility: visible;
    transform: translateX(0);
  }
}

.expand-button {
  margin-bottom: 24px;
  color: var(--text-secondary);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);

  &.expanded {
    opacity: 0;
    transform: scale(0.8);
  }

  &.collapsed {
    opacity: 1;
    transform: scale(1);
  }

  &:hover {
    color: var(--primary-main);
    transform: scale(1.1);
  }
}

.collapsed-widgets-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  align-items: center;
}

.widget-tooltip-paper {
  padding: 8px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;

  &.selected {
    cursor: not-allowed;
    opacity: 0.5;
    background-color: var(--action-disabled-background);
  }

  &:not(.selected) {
    cursor: grab;
    background-color: var(--background-paper);

    &:hover {
      background-color: var(--action-hover);
      transform: translateX(-2px);
    }
  }
}

.widget-tooltip-icon {
  display: flex;
  
  svg {
    font-size: 24px;
  }

  &.selected {
    color: var(--action-disabled);
  }

  &:not(.selected) {
    color: var(--primary-main);
  }
} 
```

```tsx
import { Box, Typography, Paper, List, ListItem, ListItemIcon, ListItemText, IconButton, Tooltip, Collapse } from '@mui/material';
import BarChartIcon from '@mui/icons-material/BarChart';
import ShowChartIcon from '@mui/icons-material/ShowChart';
import PieChartIcon from '@mui/icons-material/PieChart';
import BubbleChartIcon from '@mui/icons-material/BubbleChart';
import TimelineIcon from '@mui/icons-material/Timeline';
import ArrowForwardIosIcon from '@mui/icons-material/ArrowForwardIos';
import ArrowBackIosNewIcon from '@mui/icons-material/ArrowBackIosNew';
import { useState } from 'react';

// Define Widget interface representing each available widget
interface Widget {
  type: 'AreaChart' | 'ScatterPlot' | 'PieChart' | 'BubbleChart' | 'BarChart';
  label: string;
  icon: React.ReactNode;
  description: string;
}

interface SettingsProps {
  selectedWidgets: Set<Widget['type']>;
}

const Settings = ({ selectedWidgets }: SettingsProps) => {
  const [isExpanded, setIsExpanded] = useState(true);

  const handleDragStart = (e: React.DragEvent<HTMLDivElement>, widgetType: Widget['type']) => {
    if (selectedWidgets.has(widgetType)) {
      e.preventDefault();
      return;
    }
    e.dataTransfer.setData('widget', widgetType);
    e.dataTransfer.effectAllowed = 'move';
  };

  const widgets: Widget[] = [
    {
      type: 'AreaChart',
      label: 'Area Chart',
      icon: <TimelineIcon />,
      description: 'Visualize data trends over time with a filled area'
    },
    {
      type: 'ScatterPlot',
      label: 'Scatter Plot',
      icon: <ShowChartIcon />,
      description: 'Display relationships between two variables'
    },
    {
      type: 'PieChart',
      label: 'Pie Chart',
      icon: <PieChartIcon />,
      description: 'Show proportional data as slices of a circle'
    },
    {
      type: 'BubbleChart',
      label: 'Bubble Chart',
      icon: <BubbleChartIcon />,
      description: 'Represent three dimensions of data with size and position'
    },
    {
      type: 'BarChart',
      label: 'Bar Chart',
      icon: <BarChartIcon />,
      description: 'Compare values across categories'
    }
  ];

  return (
    <Box
      sx={{
        height: '100%',
        display: 'flex',
        flexDirection: 'row',
        position: 'relative',
        width: isExpanded ? 250 : 64,
      }}
    >
      {isExpanded ? (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            p: 2,
          }}
        >
          <Box
            sx={{
              display: 'flex',
              alignItems: 'center',
              mb: 2,
            }}
          >
            <IconButton
              onClick={() => setIsExpanded(false)}
              size="small"
              sx={{
                mr: 1,
                color: 'text.secondary',
                '&:hover': {
                  color: 'primary.main',
                },
              }}
            >
              <ArrowForwardIosIcon sx={{ fontSize: 18 }} />
            </IconButton>
            <Typography variant="h6">
              Available Widgets
            </Typography>
          </Box>

          <List
            sx={{
              flex: 1,
              overflowY: 'auto',
              '&::-webkit-scrollbar': {
                width: '8px',
              },
              '&::-webkit-scrollbar-track': {
                background: '#f1f1f1',
                borderRadius: '4px',
              },
              '&::-webkit-scrollbar-thumb': {
                background: '#888',
                borderRadius: '4px',
                '&:hover': {
                  background: '#666',
                },
              },
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <ListItem
                  key={widget.type}
                  draggable={!isSelected}
                  onDragStart={(e) => handleDragStart(e, widget.type)}
                  component={Paper}
                  sx={{
                    mb: 2,
                    cursor: isSelected ? 'not-allowed' : 'grab',
                    opacity: isSelected ? 0.5 : 1,
                    backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                    '&:hover': !isSelected ? {
                      backgroundColor: 'action.hover',
                      transform: 'translateY(-2px)',
                    } : {},
                    transition: 'all 0.2s ease',
                    borderRadius: 1,
                  }}
                >
                  <ListItemIcon sx={{ color: isSelected ? 'action.disabled' : 'primary.main' }}>
                    {widget.icon}
                  </ListItemIcon>
                  <ListItemText
                    primary={widget.label}
                    secondary={isSelected ? 'Already added to dashboard' : widget.description}
                    primaryTypographyProps={{
                      fontWeight: 'medium',
                      color: isSelected ? 'text.disabled' : 'text.primary'
                    }}
                  />
                </ListItem>
              );
            })}
          </List>
        </Box>
      ) : (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            backgroundColor: 'background.paper',
            pt: 2,
            pb: 2,
          }}
        >
          <IconButton
            onClick={() => setIsExpanded(true)}
            size="small"
            sx={{
              mb: 3,
              color: 'text.secondary',
              '&:hover': {
                color: 'primary.main',
              },
            }}
          >
            <ArrowBackIosNewIcon sx={{ fontSize: 20 }} />
          </IconButton>

          <Box
            sx={{
              display: 'flex',
              flexDirection: 'column',
              gap: 2,
              alignItems: 'center',
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <Tooltip key={widget.type} title={widget.label} placement="left">
                  <Paper
                    draggable={!isSelected}
                    onDragStart={(e) => handleDragStart(e, widget.type)}
                    sx={{
                      p: 1,
                      width: 40,
                      height: 40,
                      cursor: isSelected ? 'not-allowed' : 'grab',
                      opacity: isSelected ? 0.5 : 1,
                      backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                      '&:hover': !isSelected ? {
                        backgroundColor: 'action.hover',
                        transform: 'translateX(-2px)',
                      } : {},
                      transition: 'all 0.2s ease',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'center',
                    }}
                  >
                    <Box 
                      sx={{ 
                        color: isSelected ? 'action.disabled' : 'primary.main',
                        display: 'flex',
                        '& > svg': {
                          fontSize: 24,
                        }
                      }}
                    >
                      {widget.icon}
                    </Box>
                  </Paper>
                </Tooltip>
              );
            })}
          </Box>
        </Box>
      )}
    </Box>
  );
};

export default Settings; 
```



```tsx
import { Box, Typography, Paper, List, ListItem, ListItemIcon, ListItemText, IconButton, Tooltip, Collapse } from '@mui/material';
import BarChartIcon from '@mui/icons-material/BarChart';
import ShowChartIcon from '@mui/icons-material/ShowChart';
import PieChartIcon from '@mui/icons-material/PieChart';
import BubbleChartIcon from '@mui/icons-material/BubbleChart';
import TimelineIcon from '@mui/icons-material/Timeline';
import ArrowForwardIosIcon from '@mui/icons-material/ArrowForwardIos';
import ArrowBackIosNewIcon from '@mui/icons-material/ArrowBackIosNew';
import { useState } from 'react';

// Define Widget interface representing each available widget
interface Widget {
  type: 'AreaChart' | 'ScatterPlot' | 'PieChart' | 'BubbleChart' | 'BarChart';
  label: string;
  icon: React.ReactNode;
  description: string;
}

interface SettingsProps {
  selectedWidgets: Set<Widget['type']>;
}

const Settings = ({ selectedWidgets }: SettingsProps) => {
  const [isExpanded, setIsExpanded] = useState(true);

  const handleDragStart = (e: React.DragEvent<HTMLDivElement>, widgetType: Widget['type']) => {
    if (selectedWidgets.has(widgetType)) {
      e.preventDefault();
      return;
    }
    e.dataTransfer.setData('widget', widgetType);
    e.dataTransfer.effectAllowed = 'move';
  };

  const widgets: Widget[] = [
    {
      type: 'AreaChart',
      label: 'Area Chart',
      icon: <TimelineIcon />,
      description: 'Visualize data trends over time with a filled area'
    },
    {
      type: 'ScatterPlot',
      label: 'Scatter Plot',
      icon: <ShowChartIcon />,
      description: 'Display relationships between two variables'
    },
    {
      type: 'PieChart',
      label: 'Pie Chart',
      icon: <PieChartIcon />,
      description: 'Show proportional data as slices of a circle'
    },
    {
      type: 'BubbleChart',
      label: 'Bubble Chart',
      icon: <BubbleChartIcon />,
      description: 'Represent three dimensions of data with size and position'
    },
    {
      type: 'BarChart',
      label: 'Bar Chart',
      icon: <BarChartIcon />,
      description: 'Compare values across categories'
    }
  ];

  return (
    <Box
      sx={{
        height: '100%',
        display: 'flex',
        flexDirection: 'row',
        position: 'relative',
        width: isExpanded ? 250 : 64,
      }}
    >
      {isExpanded ? (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            p: 2,
          }}
        >
          <Box
            sx={{
              display: 'flex',
              alignItems: 'center',
              mb: 2,
            }}
          >
            <IconButton
              onClick={() => setIsExpanded(false)}
              size="small"
              sx={{
                mr: 1,
                color: 'text.secondary',
                '&:hover': {
                  color: 'primary.main',
                },
              }}
            >
              <ArrowForwardIosIcon sx={{ fontSize: 18 }} />
            </IconButton>
            <Typography variant="h6">
              Available Widgets
            </Typography>
          </Box>

          <List
            sx={{
              flex: 1,
              overflowY: 'auto',
              '&::-webkit-scrollbar': {
                width: '8px',
              },
              '&::-webkit-scrollbar-track': {
                background: '#f1f1f1',
                borderRadius: '4px',
              },
              '&::-webkit-scrollbar-thumb': {
                background: '#888',
                borderRadius: '4px',
                '&:hover': {
                  background: '#666',
                },
              },
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <ListItem
                  key={widget.type}
                  draggable={!isSelected}
                  onDragStart={(e) => handleDragStart(e, widget.type)}
                  component={Paper}
                  sx={{
                    mb: 2,
                    cursor: isSelected ? 'not-allowed' : 'grab',
                    opacity: isSelected ? 0.5 : 1,
                    backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                    '&:hover': !isSelected ? {
                      backgroundColor: 'action.hover',
                      transform: 'translateY(-2px)',
                    } : {},
                    transition: 'all 0.2s ease',
                    borderRadius: 1,
                  }}
                >
                  <ListItemIcon sx={{ color: isSelected ? 'action.disabled' : 'primary.main' }}>
                    {widget.icon}
                  </ListItemIcon>
                  <ListItemText
                    primary={widget.label}
                    secondary={isSelected ? 'Already added to dashboard' : widget.description}
                    primaryTypographyProps={{
                      fontWeight: 'medium',
                      color: isSelected ? 'text.disabled' : 'text.primary'
                    }}
                  />
                </ListItem>
              );
            })}
          </List>
        </Box>
      ) : (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            backgroundColor: 'background.paper',
            pt: 2,
            pb: 2,
          }}
        >
          <IconButton
            onClick={() => setIsExpanded(true)}
            size="small"
            sx={{
              mb: 3,
              color: 'text.secondary',
              '&:hover': {
                color: 'primary.main',
              },
            }}
          >
            <ArrowBackIosNewIcon sx={{ fontSize: 20 }} />
          </IconButton>

          <Box
            sx={{
              display: 'flex',
              flexDirection: 'column',
              gap: 2,
              alignItems: 'center',
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <Tooltip key={widget.type} title={widget.label} placement="left">
                  <Paper
                    draggable={!isSelected}
                    onDragStart={(e) => handleDragStart(e, widget.type)}
                    sx={{
                      p: 1,
                      width: 40,
                      height: 40,
                      cursor: isSelected ? 'not-allowed' : 'grab',
                      opacity: isSelected ? 0.5 : 1,
                      backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                      '&:hover': !isSelected ? {
                        backgroundColor: 'action.hover',
                        transform: 'translateX(-2px)',
                      } : {},
                      transition: 'all 0.2s ease',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'center',
                    }}
                  >
                    <Box 
                      sx={{ 
                        color: isSelected ? 'action.disabled' : 'primary.main',
                        display: 'flex',
                        '& > svg': {
                          fontSize: 24,
                        }
                      }}
                    >
                      {widget.icon}
                    </Box>
                  </Paper>
                </Tooltip>
              );
            })}
          </Box>
        </Box>
      )}
    </Box>
  );
};

export default Settings;
```
